---
sidebar: 'auto'
title: 01异步
sticky: 1
date: 2022.03.10
tags:
 - Async
categories: 
 - 09JS异步
---

# 异步

## 1.什么是异步

程序中某一操作A，现在执行，而结果需要等一段时间之后才能获取到，而在此段时间内我可以继续做其他的操作B、C、D……这种情况我们称A与其他操作之间是**异步的。**

我们称这种某一操作（A）还没执行完就可以接着执行其他操作（BCD）的现象为**“非阻塞”的。**

## 2.同步

与此相对，同步的概念则是我**必须完全处理完一件事A才能继续下一件事B**……这种串行执行我们称之为是同步的。跟此类现象相对应的一个术语叫**“阻塞”，**即一件事情执行不完就不可以继续执行接下来的事情。

## 3.异步现象

前端常见的异步事件就是请求——**向后端请求数据并接收数据。**

\- 我们理想的程序是这样执行的：

**\- 发出请求--->返回结果--->拿到数据--->利用数据做一些事情**

\- 理想的代码表示就是这样子的：

```js
var data = get('/api');
alert(data); // undefined！
```

\- 事与愿违，这里的alert(data)并不能够alert出请求返回的结果，事实上在var data = get('/api')这一句就已经没有获取到了。**原因是请求一个异步事件而获取数据和利用数据是同步的**。也就是说请求的操作get('/api')会执行一段时间，结果是在将来某一时间才会得到（我们并不能明确多久之后会得到结果），获取数据和利用数据在它执行的这段时间里就会执行。于是程序真实的执行流程是这样的：

**\- 发出请求--->拿数据（肯定拿不到啊）--->利用数据做一些事情--->返回结果**

\- 我们的大脑思维逻辑总是一件事情完了之后再去做另一件事情，尤其是两件事情相互影响关联的情况下，也可以认为这种思维是同步的。于是异步事件与我们常规的处理事件思维不同，它打乱了我们事件执行的顺序。因为我们需要拿到异步事件的结果去做一些别的事，所以会期望将非阻塞的异步转化为同步的样子来处理，**也就是说我一定要等到异步事件有了结果再执行别的事情，不要非阻塞，就要阻塞**——于是我们需要在结果返回的时刻运行我们的处理结果函数，这一运行时机的把控交给了执行环境，而我们传达这种想法给环境的方式有多种，回调函数就是其中之一。

\- 比如说让上面的代码按正确的同步流程走就可以改成：

```js
// 传入一个函数，它会在异步事件结束完成后执行（注意这里假设get方法是个第三方组装的请求方法）
get('/api', function (res) {
    var data = res;
    alert(data);
})
```

\- 重申一下异步事件：现在执行，而结果需要等一段时间之后才能获取到。换一种表达方式就是异步操作分为两步：第一发起异步任务，第二处理任务完成后得到的结果，于是这里的回调函数就充当了第二步的角色，我们在异步执行完后执行这个回调函数，那此时就肯定是能取到结果的。

## 4.异步原理

js代码片段会放入js引擎的**执行栈执行（主线程上）**，而何时执行哪一段代码则由js的宿主环境决定或者说调度。比如js在浏览器执行就是浏览器环境，在服务端执行就是node环境。

同步代码会直接放到执行栈中执行，而异步代码则会先按上面提到的第一步由主线程发起异步任务，然后交给环境里其他工作线程完成，最后取得结果时将处理结果的程序（比如回调函数）放到“**任务队列**”里，按某种调度机制再挪到执行栈上继续执行。**这些环境都提供了此调度机制，叫eventLoop（事件循环）**

### 1.执行栈

执行栈并非说是js代码执行的顺序是先进后出，这里**栈中的元素指的是执行上下文**，或者说被调用时候的某作用域，执行环境。

```js
console.log(1);
function foo() {
    console.log(2);
    bar();
    console.log(4);
}
function bar() {
    console.log(3);
}
foo();
console.log(5);
```

**\- 执行结果：1 2 3 4 5**

\- 执行栈每遇到一个执行上下文，就会将该上下文压入栈并执行该上下文里的内容。上面的代码执行流程如下图：

![img](https://gitee.com/lingisme9/images1/raw/master/img/20220310173629.png)

从其表现来看简单说就是，**主线程上执行程序会按照调用顺序依次执行**——这符合我们的预期。

同步的任务都会在这个直接在此执行栈中得到执行。

### 2.任务队列（task queue）

任务队列里存放了**处理异步事件结果的程序**，并且是当**异步事件完成后**才会将对应的处理程序入队。

**详细说就是主线程发起异步任务，工作线程完成这个异步任务得到结果，此时处理该结果的程序就会被推入任务队列等待被调用。**

-任务队列可以有一个或多个。我们常见的异步任务细究起来属于两类：

- **宏任务：setTimeout, setInterval, setImmediate, I/O操作, UI渲染**

* **微任务：process.nextTick（Nodejs）, Promises**

**这两类任务的区别（之一）是：执行的优先级是微任务大于宏任务**

### 3.事件循环

前面提到环境里负责**调度异步任务处理程序的执行顺序的机制**就是事件循环。

其实说起来比较简单，就是当主线程上**执行栈里的任务执行完**，判断为空时，主线程就会查看任务队列，然后按先进先出原则（队首）取一个任务，放到执行栈里执行，判断，再取，再执行……这个循环往复的过程就是事件循环。取一次我们称为一次tick。

\- 这个循环里还有些细节需要补充，先再重申一下相关的两点：

1. **微任务优先级大于宏任务**

2. **任务队列不止一个**



\- 所以这里要补充一下tick的细节：主线程执行栈空后会**先检查微任务排成的任务队列**，tick掉所有微任务后，在宏任务排起的队列上执行一次tick，紧接着再看向微任务队列，不断tick直至此微任务队列为空，然后再tick一个宏任务，然后再检查微任务队列……

\- 也就是说：

1. **每次tick都会先检查微任务队列**

2. **tick一次宏任务后就会转到微任务队列上一直在微任务队列上执行tick直至清空**

![img](https://gitee.com/lingisme9/images1/raw/master/img/20220310174529.png)

### 4.面试考察

\- 关于此知识点面试时的会有类似下面的考察：

```js
console.log(1);
setTimeout(()=>{
    console.log(2);
    Promise.resolve().then(ret=>{
        console.log(3);
    })
    Promise.resolve().then(ret=>{
        console.log(4);
    })
});
setTimeout(()=>{
    console.log(5);
});
Promise.resolve().then(ret=>{
    console.log(6);
})
```

1执行栈->微任务6->宏任务2->微任务3->微任务4->宏任务5

\- 问：上面输出的结果顺序是什么？

\- 答案：1 6 2 3 4 5

\- 分析如下图：

![img](https://gitee.com/lingisme9/images1/raw/master/img/20220310175047.png)

![img](https://gitee.com/lingisme9/images1/raw/master/img/20220310175220.png)

![img](https://gitee.com/lingisme9/images1/raw/master/img/20220310175243.png)

![img](https://gitee.com/lingisme9/images1/raw/master/img/20220310175300.png)

\- 以上就是关于“异步”的介绍。

\- 我们提到，异步事件和我们同步的大脑思维是不符合的，我们总是想以一种更优雅、清晰的编程方式去表达、管理异步任务，使之动向完全处于我们的掌控之内。于是js发展史中出现了几种不同的针对异步事件的编程方式。我们需要一直明确的是，**他们都一样，只是对于异步事件的不同表达，而不是什么新事物。**





































