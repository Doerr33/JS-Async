---
sidebar: 'auto'
title: 05Async/await
date: 2022.03.10
tags:
 - Async
categories: 
 - 09JS异步
---

# Async/await

## 1.基本实现原理

> async和await是生成器函数使用的语法糖，即简写版生成器：

```js
// 生成器函数写法
function *main() {
    var data = yield get('/api'); // 暂停点
    alert(data);
}
// 对应的语法糖写法
async function main() {
    var data = await get('/api'); // 暂停点
    alert(data);
}
```

\- 除了将*号替换成async，yield替换为await之外（很明显单这两点看不出来这语法糖哪里甜），更出彩的是**async/await是内置了执行器**——每遇到一个await的表达式就会自动运行并等待返回结果，然后其之后的代码也能紧接着执行，不需要next：

```js
function get(url) {
    return new Promise((resolve, reject)=>{
        ajax(url, (res)=>{
            if(res.code === 200) resolve(res);
            else reject(res);
        })
    });
}

async function main() {
    var promise = await get('/api'); // 运行get('/api')，得到的是一个promise
    return promise;
}
// 跑一下
main().then(res=>{
    alert(res.data);
})
```

\- 这样看起来怪怪的，你可能会说那我还不如直接跑get然后then——你说的很对。



## 2.语法

### 1.async关键字

通过在函数声明的前面添加async关键字，会将函数返回的结果封装为一个Promise实例对象。
返回的Promise实例对象的状态和值的规则与then方法返回的实例对象的状态和值的规则一致。

- 如果return一个非Promise实例对象值，则返回的Promise实例对象的状态为成功，且值为return 的值。
- 如果return一个Promise实例对象，则返回的Promise对象的状态和值与return的对象的状态和值一致。
- 如果抛出异常，throw，则返回的Promise实例对象的状态为失败，且值为异常信息。

### 2.await表达式

作用：获取Promise实例对象成功的结果

- await右侧的表达式一般为Promise实例对象，但也可以是其他的值
- 如果表达式是Promise实例对象，await返回的是Promise实例对象成功的值
- 如果表达式时其他的值，则直接将此值作为await的返回值。

**注意：**

- await必须写在async函数中，但async函数中可以没有await
- 如果await的右侧的Promise实例对象状态为失败，就会抛出异常，需要使用try..catch捕获异常，并且在catch中获得失败的值。

### 3.async和await结合使用

```js
const fs = require('fs');
const util = require('util');
const mineReadFile = util.promisify(fs.readFile);

async function main() {
   try {
      let f1 = await mineReadFile('../html/test.html');
      let f2 = await mineReadFile('../html/test2.html');
      console.log((f1 + f2));
   } catch (e) {
      console.log(e);
   }
}

main();
```

可以看到使用async/await语法糖可以不用手动调用then方法获取到成功/失败的结果，更加的方便。

## 3.实际应用

在实际情况中我们更可能像下面这样用：

```js
// api
function get(url) {
    return new Promise((resolve, reject)=>{
        ajax(url, (res)=>{
            if(res.code === 200) resolve(res);
            else reject(res);
        })
    }).then(res=>{
        return res.data;
    });
}

// async/await写法（常见）------------------------
async function main() {
    var data = await get('/api'); // 运行get('/api')，会等他返回了data
    alert(data);
}
// 跑一下就会直接按顺序执行，妈妈再也不用担心我异步的用同步代码取不到值或者没有拿next驱动啦
main();

// 生成器写法（既然有了语法糖我们就很少采用这种写法了）-----------------------------
function *main() {
    var data = field get('/api');
    alert(data);
}
var iter = main();
var value = iter.next().value; //推一下，得到了field后面表达式的值，是res.data
iter.next(value); //再推一下，赋值给data并且alert

// promise写法（常见）----------------------------
get('/api').then((data)=>{
    alert(data);
});
```

**即我们会在请求中把最终纯净的数据返回**







