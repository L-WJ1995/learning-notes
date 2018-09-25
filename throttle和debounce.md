# debounce和throttle
----  
近日遇到这样一个问题,滚动条拖到底部时显示Loading并进行AJAX请求,后面再渲染页面,测试时发现短时间快速得触发这个事件,页面会变得异常卡顿。(应该不只有我一个人遇到)  
其实这个问题在我很久之前看的的一本书上就有提到,javascript精解,书里面讲的是降频这个概念。在一些频率很高的事件或者连续的事件中,如果不进行性能的优化,就可能会出现页面卡顿的现象,例如:
   1. mousemove(拖曳)/mouseover(划过)/mouseWheel(滚屏)  
   2. keypress(基于ajax的用户名唯一性校验)/keyup(文本输入检验、自动完成)/keydown(游戏中的射击)    
* 在了解到这些事件可能造成的性能问题,我先想到的就是对事件触发的频率进行限制,而现在通常的应对方式正是基于这一点实现的,通常的应对方式有两种,throttle(节流)和debounce(防抖)。
* debounce和throttle的简单描述  
    1. debounce: 当事件触发一段时间后,才会执行相应的处理函数,若在这段时间间隔内又触发此事件则将重新计算时间间隔。
    2. throttle: 预先设定一个执行周期,在这个执行周期内,无论触发多少次事件,都只运行一次处理函数。  
  
## 了解debounce  
短时间内连续地移动鼠标,浏览器可能会触发几十（甚至几百）个 mousemove 事件,不使用 debounce 的话,监听函数就要执行这么多次,下面简单实现一个debounce函数,便于更加直观的了解它的功能。  
```
/**
* @param fn {Function}   实际要执行的函数
* @param delay {Number}  延迟时间
* @return {Function}     返回一个处理函数
*/
function debounce(fn, delay) {

  // 定时器,用来 setTimeout
  var timer

  // 返回一个函数,这个函数会在一个时间区间结束后的delay毫秒时执行fn函数
  return function () {

    // 保存函数调用时的上下文和参数,传递给 fn
    var context = this
    var args = arguments

    // 每次调用函数,就清除定时器,以保证不执行fn
    clearTimeout(timer)

    // 当返回的函数被最后一次调用后（也就是用户停止了某个连续的操作）,
    // 再过delay毫秒就执行fn
    timer = setTimeout(function () {
      fn.apply(context, args)
    }, delay)
  }
}
```  
debounce函数返回了一个闭包,这个闭包依然会被连续频繁地调用,但是在闭包内部,限制了原始函数fn的执行,强制fn只在连续操作停止后只执行一次。    
  
## 了解throttle
正常情况下,mousemove 的监听函数可能会每20ms（假设）执行一次,如果设置200ms的“节流”,那么它就会每200ms执行一次。比如在 1s 的时间段内,正常的监听函数可能会执行50（1000/20）次,“节流” 200ms后则会执行 5（1000/200）次,大大提升了性能,同样下面简单实现一个throttle函数。  
```
/**
* @param fn {Function}   实际要执行的函数
* @param delay {Number}  执行间隔,单位是毫秒（ms）
* @return {Function}     返回一个“节流”函数
*/

function throttle(fn, threshhold) {

  // 记录上次执行的时间
  var last

  // 定时器
  var timer

  // 默认间隔为 250ms
  threshhold || (threshhold = 250)

  // 返回的函数,每过threshhold毫秒就执行一次fn函数
  return function () {

    // 保存函数调用时的上下文和参数,传递给 fn
    var context = this
    var args = arguments

    var now = +new Date()

    // 如果距离上次执行fn函数的时间小于threshhold,那么就放弃
    // 执行 fn,并重新计时
    if (last && now < last + threshhold) {
      clearTimeout(timer)

      // 保证在当前时间区间结束后,再执行一次fn
      timer = setTimeout(function () {
        last = now
        fn.apply(context, args)
      }, threshhold)

    // 在时间区间的最开始和到达指定间隔的时候执行一次fn
    } else {
      last = now
      fn.apply(context, args)
    }
  }
}
```    
throttle的实现相比debounce多了一个时间间隔判断,其他的逻辑基本都是一致的。  
  
## 总结  
debounce强制函数在某段时间内只执行一次,throttle强制函数以固定的速率执行。  
本例中debounce和throttle的实现比较简单,完整的功能比较复杂,Lodash就提供了比较完善的功能。


