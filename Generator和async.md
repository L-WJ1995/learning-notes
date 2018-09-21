# Generator和async
----
本身async的使用比较简单,算是语法糖,了解了一下async的实现,记录一下自己的心得体会.  
  
* 实现async之前,需要先了解一下Generator函数,Generator是ES6的新增函数,名字叫生成器函数,它的大致代码如下：
    ```
    function* (){
        yield x + 1
        return/throw value
    }
    ```  
  1. Generator函数最大的特点就是可以调用函数的next()方法,让函数暂停/恢复执行,每次调用 next 方法,会返回一个对象,表示当前阶段的信息(value属性和done属性)value属性是yield语句后面表达式的值,表示当前阶段的值;done属性是一个布尔值,表示Generator函数是否执行完毕,即是否还有下一个阶段.  
  
  2. 因为可以使函数暂停/恢复执行的特性,使得它成为封装异步代码的解决方案,个人觉得算是终极解决方案,而且Generator函数提供了完善的体内外的数据交换和错误处理机制,next()方法返回的value值是函数输出的数据,next()方法可以接受一个参数,作为上一个yield表达式的返回值,这是向函数内输入数据,同时每个yield都可以部署try...catch语句,用于捕获捕获函数体外抛出的错误.  
    
  3. 下面来简单展示一下Generator封装一个异步操作.
     ```
     function get(val) {
         return new Promise(resolve => {  //这里的promise只考虑成功的情况
             setTimeout(() => {
                 console.log(val)
                 resolve(val)
             })
         })
     }

     let gen = function* () {
         let a = yield get(1)
         let b = yield get(2)
         console.log(a + b)
     }

     let g = gen()
     let result = g.next()
     result.value.then(res => {
         result = g.next(res)
         return result.value
     }).then(res => {
         result = g.next(res) //执行完本语句后 result.done === true, 结束
     })
     ```  
     上面代码中,首先执行Generator函数,获取遍历器对象,然后使用next方法(第二行),执行异步任务的第一阶段由于get返回的是一个Promise对象,因此要用then方法调用下一个next方法,最后输出结果就是 1 2 3,写的过程中就好像在写同步代码,但是封装的并不完美,流程执行不方便,而且需要手动执行,接下来封装一个自动执行函数,用来解决这个问题.
       
    4. 自动执行函数的封装  
        ```
        function run(func) {
        let g = func()

            return new Promise((resolve,reject) => { //这里返回一个Promise对象,用于自执行函数执行完之后调用then方法.
                let result = g.next()  //进入生成器函数并执行
                next()   //执行下一个异步操作(yield dosomething)
                function next() {
                    if (result.done) resolve(result.value)  //如果done为真,此时生成器函数内的所有操作已经执行完毕
                    else {
                        result.value.then(res => {
                            try{                     //捕获函数体内的错误
                                result = g.next(res) 
                            } catch (e) {
                                return reject(e)    
                            }
                            next() 
                        },rej =>{
                            try{                //捕获函数体内的错误,并向外抛出错误
                                console.log(rej)
                                result = g.next(rej) //此处可以根据需要执行g.throw(xxx)抛出错误
                            } catch (e) {
                                return reject(e)
                            }
                            next()
                        })
                    } 
                }
            })
        }

        let gen = function *(){
            let a, b, c

            try {               //用于接住异步操作中抛出的错误
                a = yield get(1)
            } catch(e) {
                throw(e)    
            }
            try {
                b = yield get(12) 
            } catch(e) {
                throw(e)
            }
            try {
                c = yield get(0)
            } catch(e) {
                throw(e)
            }

             console.log(a+b+c)

        }


        function get(val) {
            return new Promise((resolve,reject) => {
                if (val < 10){
                    setTimeout(() => {
                        console.log(val + "小于10")
                        resolve(val)
                    },1000)
                } else {
                    setTimeout(() => {
                        console.log(val + "大于等于10")
                        reject(val)
                    },1000)
                         
                }
            })
        }

        run(gen).then(()=>{console.log("执行完毕")})
        ```     
    以上的代码就是同步风格的异步代码展示,下面回到正题.
      
* 以上的代码如果使用async,如下：
  ```
    let gen = async function () {
        let a, b, c
        try {               //用于接住异步操作中抛出的错误
            a = await get(1)
        } catch(e) {
            throw(e)    
        }
        try {
            b = await get(12) 
        } catch(e) {
            throw(e)
        }
        try {
            c = await get(0)
        } catch(e) {
            throw(e)
        }

            console.log(a+b+c)
    }
  ```
  1. async函数内置执行器,而Generator函数的执行必须靠执行器,而async函数自带执行器.也就是说,async函数的执行,与普通函数一模一样.  
    
  2. 相比Generator,async的语义更好,async 表示函数里有异步操作,await 表示紧跟在后面的表达式需要等待结果.  
    
  3. async函数的await命令后面,可以跟Promise对象和原始类型的值(数值、字符串和布尔值,但这时等同于同步操作)

* 使用async函数的注意点  
  1. await命令后面的Promise对象,运行结果可能是rejected,所以最好把await命令放在 try...catch 代码块中.  
    
  2. await命令只能用在async函数之中,如果用在普通函数,就会报错.  
    
  3. 如果有多个异步操作需要并行,考虑使用Promise.all,不用使用for循环和forEach.