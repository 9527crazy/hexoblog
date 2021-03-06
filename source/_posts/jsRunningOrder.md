---
title: "js 单线程、宏任务与微任务的执行顺序"
abstract: 思考一下，js的运行
date: 2020-11-25 14:37:10
tags:
    - js
categories:
    - 前端    
---
## js 单线程
> 众所周知js是单线程，但js是可以执行同步和异步任务的，同步的任务众人皆知是按照顺序去执行的；  
而异步任务的执行，是有一个优先级的顺序的，包括了 宏任务（macrotasks）和 微任务(microtasks)

## 宏任务
> 是指消息队列中的等待被主线程执行的事件，宏任务执行时都会重新创建栈，然后调用宏任务中的函数，栈也会随着变化，但宏任务执行结束时，栈也会随之销毁。  
包括 整体代码script，setTimeout，setInterval ，setImmediate，I/O，UI renderingnew ，Promise*
## 微任务
> 可以把微任务看成是一个需要异步执行的函数，执行时机是在主函数执行结束之后、当前宏任务结束之前  
包括 Promises.(then catch finally)，process.nextTick， MutationObserver  
微任务是基于消息队列、事件循环、UI 主线程还有堆栈而来的
## 区别
> 宏任务和微任务的区别在于在事件循环机制中，执行的机制不同  
每次执行完所有的同步任务后，会在任务队列中取出异步任务，先将所有微任务执行完成后，才会执行宏任务  
所以可以得出结论， 微任务会在宏任务之前执行。  
我们在工作常用到的宏任务是 setTimeout，而微任务是 Promise.then  
注意这里是Promise.then,也就是说 new   Promise在实例化的过程中所执行的代码是同步的，而在then中注册的回调函数才是异步。

``` javascript
setTimeout(function(){
    console.log('1')
});
new Promise(function(resolve){
    console.log('2');
    resolve();
}).then(function(){
    console.log('3')
});
console.log('4');
new Promise(function(resolve){
    console.log('5');
    resolve();
}).then(function(){
    console.log('6')
});
setTimeout(function(){
    console.log('7')
});
function bar(){
    console.log('8')
    foo()
}
function foo(){
    console.log('9')
}
console.log('10')
bar()
```
## 解析：  
>首先浏览器执行Js代码由上至下顺序，遇到setTimeout，把setTimeout分发到宏任务Event Queue中  
new Promise属于主线程任务直接执行打印2
Promis下的then方法属于微任务，把then分到微任务 Event Queue中  
console.log(‘4’)属于主线程任务，直接执行打印4  
又遇到new Promise也是直接执行打印5，Promise 下到then分发到微任务Event Queue中  
又遇到setTimouse也是直接分发到宏任务Event Queue中，等待执行
console.log(‘10’)属于主线程任务直接执行  
遇到bar()函数调用，执行构造函数内到代码，打印8，在bar函数中调用foo函数，执行foo函数到中代码，打印9  
主线程中任务执行完后，就要执行分发到微任务Event Queue中代码，实行先进先出，所以依次打印3，6  
微任务Event Queue中代码执行完，就执行宏任务Event Queue中代码，也是先进先出，依次打印1，7。  
最终结果：2，4，5，10，8，9，3，6，1，7
