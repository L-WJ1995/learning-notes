# Event Loop 事件循环  
----
首先要明确:  
**JS是单线程语言**,也就是说，JS一次只能做一件事情。  
cpu处理指令速度非常快，远比磁盘I/O和网络I/O速度快，所以一些cpu直接执行的任务就成了优先执行主线任务（即同步任务synchronous），然后需要io返回数据的任务就成了等待被执行的任务（即异步任务asynchronous）  
同步任务：在主线程上排队执行的任务，前一个任务执行完毕，才能执行后一个任务；  
异步任务：不进入主线程、而进入”任务队列”（task queue）的任务，只有”任务队列”通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。  
所以：
**当要主线程任务完成会后，就会去读取”任务队列”，这就是JavaScript的运行机制**  
  
## Microtasks 和 Macrotasks  
具体到任务队列，又分为 microtasks(微任务) 和 macrotasks(宏任务)  
属于microtasks的任务有：  
1. process.nextTick  
2. promise  
3. Object.observe  
4. MutationObserver  
  
属于macrotasks的任务有：  
1. setTimeout  
2. setInterval  
3. etImmediate  
4. I/O  
5. UI渲染  
  
在执行事件时：  
1. 从script(整体代码)开始第一次循环。执行所有主线程的同步函数，遇到异步函数，分别添加到microtasks和macrotasks任务队列.    
2. 同步函数执行后，开始执行异步函数中的任务队列，首先执行所有的micro-task  
3. 所有的micro-task执行完成后，循环执行macro-task任务  
4. macro-task任务执行，再次执行micro-task，这样一直循环下去  
  
简单来件，整体的js代码在执行完主线程的同步任务，然后有microtask执行microtask，没有microtask执行下一个macrotask，如此往复循环至结束
  
## 结尾附两道经典题
```
setTimeout(function() {
  console.log(4)
}, 0);
new Promise(function(resolve) {
  console.log(1)
  for (var i = 0; i < 10000; i++) {
    i == 9999 && resolve()
  }
  console.log(2)
}).then(function() {
  console.log(5)
});
console.log(3);

// 结果是1 2 3 5 4
```  
  
```
console.log('start')
const interval = setInterval(() => {
  console.log('setInterval')
}, 0)
setTimeout(() => {
  console.log('setTimeout 1')
  Promise.resolve().then(() => {
    console.log('promise 3')
  }).then(() => {
    console.log('promise 4')
  }).then(() => {
    setTimeout(() => {
      console.log('setTimeout 2')
      Promise.resolve().then(() => {
        console.log('promise 5')
      }).then(() => {
        console.log('promise 6')
      }).then(() => {
        clearInterval(interval)
      })
    }, 0)
  })
}, 0)
Promise.resolve().then(() => {
  console.log('promise 1')
}).then(() => {
  console.log('promise 2')
})

// 执行顺序是
// start
// promise 1
// promise 2
// setInterval
// setTimeout 1
// promise 3
// promise 4
// setInterval
// setTimeout 2
// promise 5
// promise 6
```