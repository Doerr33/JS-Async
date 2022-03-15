---
sidebar: 'auto'
title: 04生成器Generator
date: 2022.03.10
tags:
 - Async
categories: 
 - 09JS异步
---

# 生成器Generator

## 1.生成器

> 你可能会问这个生成器能生成什么？答案是能生成迭代器（Iterator）。

### 1.Iterator迭代器

**迭代器，也叫遍历器，是为了以统一方式遍历某类型的某组数据时需要的访问接口**，说白了就是，只要一组数据拥有迭代器，就能够以for...of...的形式遍历里面的元素。

 比如Array类型，Map类型，Set类型能以for...of...的形式遍历里面的元素，因为他们都有默认的迭代器接口，即他们身上有一个属性叫Symbol.iterator，是构造器的构造函数；**而普通Object对象则不能，因为它没有迭代器接口。**



\- 某一组数据的迭代器上有next的方法，可以手动挨个获取元素：

```js
// 一个可以迭代的数组
var arr = [5,4,3,2,1];
// 实例化它的一个迭代器
var iter = arr[Symbol.iterator]();
iter.next(); // {value: 5, done: false}
iter.next(); // {value: 4, done: false}
iter.next(); // {value: 3, done: false}
```

### 2.简单认识Generator

**Generator最大的特点就是能控制程序的运行与暂停**，就像debug一样：打一个断点，然后可以控制程序按我们的意愿走走停停。

```js
// 加了*号表示这是一个生成器函数
function *foo() {
    var val = 1;
    // 进入此循环val就会不断加1
    while(true) {
        val ++;
        // 碰到yield的地方会暂停
        var temp = yield val;
        console.log(temp);
    }
}
// 调用生成器函数能生成一个迭代器，而不执行生成器函数里的逻辑
var iter = foo();

// 每次next就会执行到下一个yield停止，并返回一个对象表示迭代器里的值（value,表示yield后面式子的值，这里就是val的值）与是否迭代完成（done）

iter.next(); //{value: 2, done: false}
// yield没有返回值，temp赋为undefined
iter.next(); //{value: 3, done: false} undefined
// 传递给next方法的参数会被当做yield的返回值，赋值给temp
iter.next(55); //{value: 4, done: false} temp=55
```

## 2.Generator控制异步

我们说Generator又是一种异步编程方案，看了上面的你可能还不能联想到这跟异步有什么关系，继续拿回调函数方案做对比：

```js
// 我们要实现的流程是：请求数据->成功：alert / 失败：throw抛错

// 一个看起来比较优雅的回调实现
function get(url, cb) {
    ajax(url, cb);
};
// 调用
get('/api', (res) => {
    if(res.code === 200) alert(res.data);
    else throw res;
});
// ---------------------------------------------
// 用生成器实现
function get(url) {
    ajax(url, (res) => {
        if(res.code === 200) iter.next(res.data);
        else iter.throw(res);
    });
}
function *main() {
    // 请注意这两句熟悉的代码
    var data = yield get('/api'); // 这里是个暂停点
    alert(data);
}
// 得到迭代器
var iter = main();
// 请求，启动！跳到get函数里，如果200就会继续iter.next(res.data)，然后执行yield后面的alert(data);
it.next();
```

一开始我们讨论异步时提到过下面错误的代码：

```js
var data = get('/api');
alert(data); // undefined！
```

 然后我们这里利用生成器让这样的同步代码正确执行了异步：

```js
function *main() {
    var data = yield get('/api');
    alert(data); // data取到了值
}
```

\- 用生成器写异步最大的好处就在于此，**把异步事件按同步思维表达出来**，代码读起来非常舒适。

\- 这种方案可以理解为在回调方案里加入了生成器（callback+Generator的编程方案）。

\- 用图来描述一下程序执行流程：

![img](https://gitee.com/lingisme9/images1/raw/master/img/20220310190545.png)



\- 这里能正确执行的**秘诀就在于我们的yield“断点”阻止了异步事件完成之前给data赋值以及alert**，**而是生成器之外的get方法不会被暂（暂停的只是生成器体内的语句执行）**，在异步函数的结果处理函数里解除暂停，而此结果处理函数是在请求结束返回结果时才会执行，进而保证了给data赋值以及alert的操作会在得到结果时执行。**概而言之，就是把非阻塞的异步转化为阻塞的同步了**

**\- 于是程序执行的顺序是：var data -> ajax('/api' -> 调用回调得到值) -> 给data赋值 -> alert(data)**

## 3.Generator控制异步的缺点

但是，**生成器的使用虽然完美地解决了回调编程里代码执行顺序混乱难读的问题，我们发现它并没有解决信任问题**：这里依旧出现了控制反转，把对结果的处理权交给了get方法。而使用promise虽然解决了信任问题但代码又不如Generator来的好看。

\- 怎么办呢？既然Generator和promise都有各自的优点，那我们就各取所长，将二者结合使用不就好了~

## 4.Generator + Promise组合控制

\- 先再用promise方式写一下上面的请求例子：

```js
function get(url) {
    return new Promise((resolve, reject)=>{
        ajax(url, (res)=>{
            if(res.code === 200) resolve(res);
            else reject(res);
        })
    });
}
get('/api').then(
    function resolved(res)=>{
        alert(res);
    },
    function rejected(err)=>{
        throw err;
    }
)
```

\- 我们非常中意它的状态转换机制，因为这解决了信任问题，但我们还想要Generator优雅的同步表达。于是我们在promise中加入了Generator：

```js
function get(url) {
    return new Promise((resolve, reject)=>{
        ajax(url, (res)=>{
            if(res.code === 200) resolve(res);
            else reject(res);
        })
    });
}
// 生成器函数
function *main() {
    var data = yield get('/api'); // 暂停点
    alert(data);
}
// 得到迭代器
var iter = main();
var promise = iter.next().value; // 取得yield后面表达式的值，这里就是get()返回的promise

// 根据promise的执行结果状态决定生成器函数里的下一步行动，推动promise和生成器的执行
promise.then(
    // 成功返回值那就继续生成器函数（为data赋值和alert）
    function resolved(res)=>{
        iter.next(res.data);
    },
    // 失败就抛错
    function rejected(err)=>{
        iter.throw(err);
    }
)
```

\- 可以看到我们的生成器函数依旧是优雅的同步代码，而异步代码我们已经用可信任的promise替换掉了。



\- 整体的执行流程和回调+Generator类似：

**\- var data -> ajax('/api' -> 得到不可再变的成功状态) -> 给data赋值 -> alert(data)**

## 5.组合控制的缺点和解决方法

\- 上面例子有点恶心的地方在于，我们需要在优雅的生成器函数的背后，手动推动迭代器next下去，编写并调用适用于promise的then（链）方法以期为生成器函数提供需要的promise（[iter.next](https://link.zhihu.com/?target=http%3A//iter.next/)().value）。

\- 信任问题交给promise解决了，我们只想舒畅地使用优雅的生成器函数，不想写推动promise一直then下去的的那一坨代码——如果能让他自己跑完promise链把最后的结果通过next抛给main函数就好了！

\- 很多人都想到了这个问题，于是有很多相关库实现了这种自动化工具，比如下面这段代码就是其中之一：

```js
// 摘自《你不知道的JS》（中卷）

function run(main) {
    // args取到了可能要传给生成器函数的参数
    var args = [].slice.call(arguments, 1), iter;
    // 调用生成器函数得到迭代器
    iter = main.apply(this, args);
    // 返回一个promise
    return Promise.resolve().then(
        // 推动main函数继续执行
        function handleNext(data) {
            var next = iter.next(data);
            return (function handleResult(next) {
                // 执行完了吗
                if(next.done) {
                    return next.value;
                } else {
                    // 没有执行完就继续
                    return Promise.resolve(next.value).then(
                        handleNext,
                        function handleErr(err) {
                            return Promise.resolve(
                                it.throw(err);
                            ).then(handleResult);
                        }
                    )
                }
            })(next);
        }
    )

}
```

\- 如果你不需要自定义化把控不同状态的promise的回调而只关注最终数据结果，那就尽情将过程交给此类工具（叫执行器），还有更为复杂精妙的工具（比如co库），我没用过，感兴趣可以去了解，你也可以自己封装。

\- 利用这个工具我们的代码就可以更加优雅：

```js
......
function *main(){
    ......
}
run(main);
```



## 6.迭代器和生成器详解







































