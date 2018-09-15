# Promise使用中需要注意的点
----  
1. Promise对象的链式调用,举例如下代码：
    ```
    function get(url) {
        return new Promise((f,g) => {
            let xhr = new XMLHttpRequest()
            xhr.open("GET",url)
            xhr.onload = () => {
                if (xxxx) f(xhr.responseText)
                else g(new Error("failed")) 
            }
            xmr.send()
        })
    }

    //这里只考虑resolve的情况。
    get("a.json").then(json_a => {
        return get("b.json").then(json_b => {
            return get("c.json").then(json_c => {
                 xxx()
            })
        })
    })
    ```
    以上的代码,获取a.json成功调用获取b.json,再成功继续获取c.json。写习惯了callback很容易写成这种形式,本身没什么错,但是PromiseAPI给我们提供了便捷的链式调用,promise.then()根据调用者返回的promise的对象的对象确定自身执行状态,所以以上代码可以写成这种形式:
    ```
    get("a.json").then(json_a => {
        return get("b.json")
    }).then(json_b => {
        return get("c.json")
    }).then(json_c => {
        xxx()
    })
    ```  

2.  Promise.then()调用时的函数执行只关心调用者是否正常返回,哪怕返回的是undefined也是成功,抛出错误则状态为失败,这里要注意一点,如果返回的是一个promise对象,例如:
    ```
    let a = new Promise((resolve,reject) => {
        setTimeout(() => reject(), 5000)
    })
        
    let b = new Promise((resolve,reject) => {
        setTimeout(() => resolve(a), 1000)
    })
    b.then(() => console.log("aaa"),() => console.log("bbb"))
    ```
    输出了"bbb",这和认知不同,b执行了resolve,应该是log出"aaa"才对。这就是需要注意的地方,promise执行过程中引入了另一个promise对象,这里创建了两个promise对象,b在1秒时resolve(a),而此时a的状态未定,所以p.then()就在等待着a返回执行状态,直到a执行reject状态。promise标准规定一个promise对象来resolve/reject另一个promise，这两个promise就会自动关联起来,a作为b的状态返回,本例中a在第五秒时状态变为reject,所以b.then执行reject方法。  

3.  有别于Promise.all的串行异步
    ```
    function get(val) {
        return new Promise(resolve => setTimeout(() => resolve(console.log(val)), 1000))
    }

    let ary = [0,1,2,3,4,5,6,7,8,9]

    ary.forEach((item) => get(item))
    ```
    以上代码大致的期望就是串行执行promise，每隔1s输出一个数值,如果按照以上的代码,实际会在1s后同时输出所有数值,原因在于一个Promise对象在创建的时候就立即执行了,如果修改成如下的代码,就会按照期望的执行顺序输出。  
      
    ```
    Promise.resolve()
    .then(() => get(0))
    .then(() => get(1))
    .then(() => get(2))
    ......
    .then(() => get(9))
    .then(() => console.log("all done"))
    ```
    可以看到就是在用链式调用,前面一个Promise执行完返回状态后,开始下一个,这里可以使用reduce方法来优化以上的代码  

    ``` 
    ary.reduce(function(item, val) {
        return item.then(function() {
            return get(val)
        })
    }, Promise.resolve())
    .then(()=>console.log("all in"))
    ```

4.  Promise.race,我认为用容错来理解更加合适一点,有如下代码：
    ```
    let p1 = new Promise(resolve => {
        setTimeout(() => {
            xxxxx
            xxxx
            resolve(a)
        })
    });
    let p2 = new Promise(resolve => {
        setTimeout(() => {
            xxxxx
            xxxx
            resolve(b)
        })
    });
    Promise.race([p1, p2]).then(function (result) {
        console.log(result);
    });
    ```
    如果p1先返回状态,那么Promise.then就响应p1的返回,p2则被抛弃,反之亦然。  
      
5. 养成使用catch接住错误的习惯,Promise执行过程中如果发生异常,如果没有reject接住并且也没有catch,那么任何抛出的异常都会被吃掉,外部Promise对象和try catch都无法捕获,Debug的时候会非常的麻烦,所以养成使用catch的习惯。

