---
sidebar: 'auto'
title: 03期约Promise
date: 2022.03.10
tags:
 - Async
categories: 
 - 09JS异步
---

# 期约Promise

> 上面提到回调函数方式的**缺点是缺乏顺序性和可控性**，于是我们又推出了Promise来进行异步编程，以期弥补回调的缺点。

## 1.promise原理

### 1.基本实现原理

我们将**异步事件包装为promise**，执行完后仍返回的是一个promise对象。**使用promise我们可以监控到异步事件的执行状态**：发起一个异步事件，没有结束时为pending状态，任务失败后为rejected状态，成功返回结果后为resolved，并且一旦这个状态由pending转换为成功或者失败状态后就是**不可再发生变化的**。

\- 我们使用.then的方式获取到它的执行结果（是个promise），并用两个函数作为then方法里的参数表示成功状态的处理方法和失败状态的处理方法。

![img](https://gitee.com/lingisme9/images1/raw/master/img/20220310180716.png)

### 2.可信任

\- 区别于用回调表示异步的方式，回调函数是从异步事件开始到处理结果全部交给第三方完成，**而promise最终只会返回异步事件的最终执行的状态而不自行处理结果**，于是对于处理结果的方法的控制权仍在我们手里，我们想如何调用，何时调用就怎么调用，**并且promise状态一旦发生改变就不会再发生变化，意味着某状态的回调只会执行一次——这便是可信任得了。**

```js
// 回调的写法（假设get函数接收三个参数，后两个是成功和失败分别调用的回调）
// 我们并不知道get这个任务会什么时候调用、调用几次我们的回调（是由get函数内部实现控制的）。
get('/api', function (res) {
    alert(res);
}, function (err)=>{
    alert(err);
});


// -----------------------------------------------------------
// promise写法（get方法返回一个只变化一次状态的promise）
function promise(url) {
    return new Promise((resolve, reject)=>{
        // new Promise里的函数会立即执行，结果在then函数里取
        get(url, (res,err)=>{
            // 状态只会变化一次，很安全
            if(err) reject(err);
            else resolve(res);
        })
    });
}


// promise('/api')得到一个promise，执行then之后又返回一个promise
promise('/api').then(
    // 如果该任务执行成功我们规定调用这个resolved函数，否则执行rejected函数
    function resolved(res)=>{
        alert(res);
    },
    function rejected(err)=>{
        alert(err);
    }
)
```

### 3.串行的链式调用

我们之前提到promise执行一次后返回的仍是promise，并且我们使用then方法接收处理promise。也就是意味着如果每个得到的promise中都有下一步操作的promise的话我们可以一直then下去形成一条链，并且他们的执行顺序是肯定和我们写出来的then链保持一致。

\- 对比之前回调函数的“回调地狱”:

```js
// 回调嵌套的情况用回调函数写出来既不美观可读性也不高
fn1(function callbackA(data){
    fn2(function callbackB(data){
        fn3(function callbackC(data){
            ......
        });
    });
});


// 封装一个异步操作为promise
var fn1 = new Promise((resolve, reject)=>{
    var data = ajax('./api', (res,err)=>{
        if(err) reject(err);
        else resolve(res);
    })
})

// then的链式调用写出来就很符合我们的思维，宛如同步代码一般，令猿舒适，而实际含义和上面的嵌套代码相同。
fn1.then(function callbackA(res){
    resolve(res);
}).then(res=>{
    fn2(function callbackB(res){
        resolve(res);
    });
}).then(res=>{
    fn3(function callbackC(res){
        ......
    });
})
```

## 2.语法

### 1.promise的状态改变

**promise的状态指的是：promise实例对象中的一个属性：【PromiseState】，一共有三个属性值**

- pending 未决定的，初始化时的默认值
- resolved / fullfilled 成功
- rejected 失败

**promise的状态只可能发生以下几种情况：**

- pending => resolved
- pending => rejected

**且状态一旦发生变化，就不可逆。**

```js
let pro = new Promise(function (resolve, reject) {});
console.log(pro);
```

在浏览器环境中进行查看，node环境只能查看状态，所以无法看到对象的其他属性值

![img](https://gitee.com/lingisme9/images1/raw/master/img/20220310182102.webp)

**如何更改promise状态呢？**

可以通过传给Promise构造函数的初始化函数中的参数resolve/reject来改变状态

- resolve: pengding=>fulfilled/resolved
- reject: pengding=> rejected
- throw错误: pending=>rejected

### 2.promise对象的值

promise对象的值指的是：实例对象中的一个属性：**【PromiseResult】，该属性存储的是异步任务成功/失败的结果。**
能修改该属性的方法只有两个：

- resolve
- reject

**通过以上方式修改【PromiseResult】的值，后期可以通过then/catch方法的调用获取到该属性的值。**

### 3.promise工作流程

```js
graph TD
A[new Promise--pending状态] 
A --> C{执行异步操作}
C -->|成功 执行resolve| D[修改promise状态为resolved]
C -->|失败 执行reject| E[修改promise状态为rejected]
D -->F[执行then方法中第一个回调函数]
E -->G[执行then方法中第二个回调函数或者执行catch方法中回调函数]
F -->H[以上方法执行完成之后 返回一个新的promise实例对象]
G-->H
```

### 4.Promise API

#### 1.初始化Promise

```js
let pro = new Promise(excutor)
```

其中：**excutor为执行器函数，执行器会在Promise内部立即 同步调用，而异步操在执行器中执行。**

什么意思呢？

**当js执行到当前行的时候，excutor函数内部有同步代码，则会立即执行，而如果遇到了异步操作的代码，则会放在任务队列中。**

除此之外，执行器默认接收两个参数，一个是resolve函数，一个是reject函数，开发者可以利用这两个参数在异步任务成功/失败的时候进行调用。

#### 2.Promise.prototype.then()

该方法会在异步任务执行完成，且异步任务中调用了resolve/reject，同时该任务处于任务队列中最头部位置时，进行调用。

接收两个参数：

- onResolved：异步任务成功时的回调函数
- onRejected：异步任务失败时的回调函数

该方法会返回一个新的Promise实例对象。

**那么返回的新的Promise实例对象的状态以及值是如何指定的呢？**

以下不管是成功还是失败回调函数都是一样的规则

- 如果回调函数中抛出异常(throw)，则返回一个失败的Promise实例对象，值为抛出异常的描述信息。
- 如果回调函数中return一个非Promise对象值，则返回一个成功的Promise实例对象，值为return的值。
- 如果回调函数中return一个Promise对象值，状态和值由Promise对象的状态和值决定。

**注意：如果回调函数中既没有throw，也没有return一个值，那么返回的新的Promise实例对象状态依然是resolve，但是其值为undefined**

```js
let p = new Promise((resolve, reject) => {
    reject(12);
});

let re = p.then(value => {
    console.log(value);
}, reason => {
    return Promise.resolve('0')
});

console.log(re); // 【PromiseState】: resolve 【PromiseResult】: 0;
```

**链式调用**:

由于then方法是可以指定Promise实例中异步任务成功/失败时的回调函数，那也就是对任何的Promise实例对象都可以调用then方法对这两种状态进行处理。从以上的实例可以看出，then方法会返回一个新的Promise实例对象，所以我们可以通过链式调用来处理需要并联进行的操作。
比如：

```js
let p = new Promise((resolve, reject) => {
    reject(12);
});
p.then(value => {
    console.log(value);
}, reason => {
    return Promise.resolve('0')
}).then(value => {
    console.log(value);// 0
}).then(value => {
    console.log(value);// undefined
});
```

#### 3.Promise.prototype.catch()

该方法会在**异步任务失败**时进行调用，属于一个语法糖，其内部实现还是使用了then方法
该方法会返回一个新的已解决的Promise实例对象。

**需要注意的是，以上对失败的处理(then的第二个参数/catch)，都是异步处理的，不能使用同步的方法try/catch进行处理**

#### 4.Promise.resolve()

该方法属于Promise构造函数，而不属于Promise实例对象，调用该方法可以立即得到一个Promise的实例对象。该实例对象的状态采用以下规则确定：

1. 如果传入的参数为 **非Promise类型的对象**，则返回的对象为成功状态的Promise实例对象
2. 如果传入的参数为 **Promise类型的对象**，则返回的对象状态根据传入的参数的状态来决定。

```js
const p = Promise.resolve(new Promise((resolve,reject)=>{
    reject('1');
}))
```

则p的【PromiseState】的值为: Rejected。

#### 5.Promise.reject()

类似于Promise.resolve()，也是属于Promise构造函数的方法，但是这个方法，总是获得一个失败的Promise期约，且传递的值作为该方法的返回值，不管传递什么参数都是如此。它不同于Promise.resolve对参数类型不同返回的期约是不同的。

比如：传入一个已经解决的期约，返回的是一个失败的期约，且其值为已解决的期约。

```js
let pro = Promise.reject(new Promise(function (resolve, reject) {
        resolve('ok');
}))
console.log(pro);
```

![img](https://gitee.com/lingisme9/images1/raw/master/img/20220310183352.webp)

#### 6.Promise.all()

是对多个期约作用的一个方法。
**参数：**期约的集合
**返回状态(【PromiseState】)**：当且仅当所有期约都是已解决的期约，才返回已解决的期约。只要有一个期约为失败的期约，那么返回一个失败的期约。
**返回的结果值(【PromiseResult】)**：当且仅当所有期约都是已解决的期约，结果值为所有已解决的结果值的集合。只要有一个期约为失败的期约，结果值为：最开始返回失败期约的结果值。

**例子（失败案例）：**

```js
let p1 = Promise.resolve('1');
let p2 = Promise.reject('ii');
let p3 = Promise.reject('ok');

let p4 = Promise.all([p1,p2,p3]);
console.log(p4);
```

![img](https://gitee.com/lingisme9/images1/raw/master/img/20220310183510.webp)

**例子（成功案例）：**

```js
let p1 = Promise.resolve('1');
let p2 = Promise.resolve('ii');
let p3 = Promise.resolve('ok');

let p4 = Promise.all([p1,p2,p3]);
console.log(p4);
```

![img](https://gitee.com/lingisme9/images1/raw/master/img/20220310183601.webp)

#### 7.Promise.race()

类似于Promise.all()，也是一个对多个期约作用的方法。
**参数：**期约的集合
**返回状态(【PromiseState】)**：由第一个返回的期约的状态所决定。
**返回值(【PromiseResult】)：**由第一个返回的期约的返回值所决定。

```js
let p1 = new Promise(function (resolve, reject) {
        setTimeout(() => {
            resolve(1);
        }, 1000);
})

let p2 = Promise.reject(2);
let p3 = Promise.resolve(3);

let p4 = Promise.race([p1,p2,p3]);
console.log(p4);
```

![img](https://gitee.com/lingisme9/images1/raw/master/img/20220310183653.webp)

由于第一个期约需要等待1s之后，才能返回，而第二个期约能够直接返回，所以期约状态为第二个的状态，期约返回值为第二个的返回值。

## 3.常见问题

### 1.一个promise指定多个成功/失败回调函数，都会调用么？

当promise对象状态发生变化时，对应状态的回调函数都会被调用。

```js
let p = new Promise((resolve, reject) => {
    resolve(12);
});

p.then(value => {
    console.log(value);
});

p.then(value => {
    console.log(value);
});
```



![img](https://gitee.com/lingisme9/images1/raw/master/img/20220310183805.webp)



可以看到，当promise状态改变为成功时(resolve)，指定的两个成功回调函数(then的第一个参数)，都被调用了。

### 2.改变promise状态和指定回调函数谁先谁后？

1. 都有可能，正常情况下是先指定回调再改变状态，但也可以先改状态再指定回调。**这里的指定回调指的是：初始化回调函数(不一定执行)**
2. **什么情况下哪个事件先发生呢？**

- 先改状态再指定回调

  1. 执行器函数中执行的是同步任务，同步任务直接调用了resolve/reject
  2. 延迟更长时间才调用then

- 先指定回调再改变状态

  执行器函数中执行的异步任务。

从上面我们总结的情况可以看出：由于js执行机制的原因，先执行同步任务，遇到异步任务放到异步队列中，等同步任务执行完毕，再通过事件循环执行异步任务。**所以如果是在同步任务中直接调用resolve/reject，状态就会立即改变，接着再初始化回调函数。但是如果是在异步任务中调用resolve/reject，那么异步任务会在同步任务之后执行，此时就会先初始化回调函数，再改变状态。**

3. **那什么时候获取的数据呢？**

- 如果先指定回调，当状态改变时，回调函数就会调用，获取到数据
- 如果先改变状态，当指定回调函数的同时，回调函数而会被调用，得到数据。

### 3.promise异常传透

当使用promise的then链式调用时，可以在最后指定失败的回调。前面任何操作出了异常，都会传到最后失败的回调中处理。但是这里有个前提，如果一旦该异常出现后，在最后失败回调之前调用了失败回调，则该异常就会被捕获。

```js
let p = new Promise((resolve, reject) => {
    reject(12);
});

p.then(value => {
    console.log(value);
}, reason => {
    console.log(reason);// 异常被捕获了
}).then(value => {
    console.log(value);
}).catch(reason => {
    console.log(1);
    console.log(reason); // 无法到达这里。
});
```

我们也可以统一处理异常，也即在最后添加失败回调。

```js
let p = new Promise((resolve, reject) => {
    reject(12);
});

p.then(value => {
    console.log(value);
}).then(value => {
    console.log(value);
}).catch(reason => { // 这里也可以使用then的第二个参数
    console.log(reason); // 捕获到了异常
});
```

### 4.中断Promise链

Promise链：利用Promise实例对象可以链式调用then/catch的特性所形成的链。
**需求：**当仅仅想执行当且回调函数，不想再执行之后的链式调用。
**做法：**返回一个pending状态的Promise实例对象。

```js
let p = new Promise((resolve, reject) => {
    resolve(12);
});

p.then(value => {
    console.log(value);
    return new Promise(() => {});
}).then(value => {
    console.log(value);
}).catch(reason => {
    console.log(reason);
});
```

**为什么返回pending状态的Promise实例对象可以实现呢？原因：Promise实例一旦发生变化(pending->resolve/reject)，都会去调用与之对应的回调函数，也即总会去执行then/catch方法。而pending状态不会发生以上状况。**

## 4.使用Promise封装异步操作

### 1.promise封装：fs模块

```js
const fs = require('fs');
let promise = new Promise(((resolve, reject) =>  {
    fs.readFile('../html/test.html', (err, data) => {
        if (err) reject(err);
        resolve(data);
    })
}))

promise.then(data => {
    console.log(data.toString());
}, reason => {
    console.log(reason);
})
```

### 2.node中自动创建封装(promisify)

> 用这个就不用上面的手动封装了

node中util模块里面的函数：promisify
**该函数的作用：**将异步函数自动进行Promise封装，并且返回一个Promise实例，这样就不需要以上方式手动封装。
**使用方法：**

- 引入util模块

```js
const util = require('util');
```

- 将待封装的异步操作传入到promisify函数中

```js
let mineReadFile = util.promisify(fs.readFile);
```

**注意：此处不需要传入异步操作的参数**

- 使用已封装的函数，使用方法和原来异步操作函数一致（只是对其进行了封装而已）

```js
mineReadFile('../html/test.html')
   .then(data => {
       console.log(data.toString());
   }, reason => {
       console.log(reason)
   })
```

- 完整的代码

```js
const fs = require('fs');
const util = require('util');

let mineReadFile = util.promisify(fs.readFile);

mineReadFile('../html/test.html')
    .then(data => {
        console.log(data.toString());
    }, reason => {
        console.log(reason)
    })
```

## 5.手写一个Promise

```js
class Promise {
    constructor(executor) {

        this.PromiseState = 'pending';
        this.PromiseResult = null;

        // 保存异步任务的回调函数
        this.callbacks = [];

        // 保存当前实例的上下文
        const self = this;

        function resolve(data) {
            // 判断状态，保证状态只会被修改一次
            if (self.PromiseState !== 'pending') return;
            // 1. 改变状态
            self.PromiseState = 'fulfilled';
            // 2. 改变值
            self.PromiseResult = data;

            // 处理异步任务的结果的回调
            // 异步处理结果
            setTimeout(() => {
                self.callbacks.forEach(item => {
                    item.onResolved(data);
                })
            })
        }

        function reject(data) {
            // 判断状态，保证状态只会被修改一次
            if (self.PromiseState !== 'pending') return;
            // 1. 改变状态
            self.PromiseState = 'rejected';
            // 2. 改变值
            self.PromiseResult = data;

            // 处理异步任务的结果的回调
            // 异步处理结果
            setTimeout(() => {
                self.callbacks.forEach(item => {
                    item.onRejected(data);
                })
            })
        }

        // 处理throw的方法
        try {
            executor(resolve, reject);
        } catch (e) {
            // 改为失败的状态
            reject(e);
        }
    }

    // 添加then方法
    then(onResolved, onRejected) {
        // 保存上下文
        const self = this;
        // 判断回调函数参数，如果不是函数，则给其一个默认值，实现异常穿透，等效于帮用户将异常传递到最后的异常捕获的回调函数中进行处理
        // 如果没有这一步，当该实例reject/throw的时候，且失败处理放在最后，那么之前的then方法都会因为onRejected为undefined而抛出异常
        // 虽然最后还是会被catch捕获，但是失败值会是：onRejected is not a function，而不是期待输出的失败值。
        if (typeof onRejected !== 'function') {
            onRejected = reason => {
                throw reason;
            }
        }

        // 处理如果then方法中什么参数都没有传递的情况，防止报错。注意：错误的创建都需要用户去定义，而不是源码本身去创建用户不知道的错误。
        if (typeof onResolved !== 'function') {
            onResolved = value => value;
        }
        // 返回一个新的Promise实例
        return new Promise((resolve, reject) => {
            // 封装函数
            function callback(type) {
                // 处理throw的情况
                try {
                    // 获取回调的执行结果
                    let result = type(self.PromiseResult);

                    // 如果是Promise实例，新的Promise实例对象则需要根据当前Promise实例状态来指定
                    if (result instanceof Promise) {
                        // 由于result是Promise实例对象，所以也可以调用then方法，根据result实例对象的状态会调用对应的回调函数
                        result.then(value=> {
                            resolve(value);
                        }, reason => {
                            reject(reason);
                        })
                    } else {
                        // 不是Promise实例对象，则直接将新的Promise实例对象状态设为成功
                        resolve(result);
                    }
                } catch (e) {
                    // 如果抛出异常，则将状态改为失败【rejected】
                    reject(e);
                }
            }
            // 以下只针对同步方式
            // 成功执行回调，并把成功结果传入
            if (this.PromiseState === 'fulfilled') {
                // then方法是异步任务的特性
                setTimeout(() => {
                    callback(onResolved);
                })
            }
            // 失败执行回调，并把失败结果传入
            if (this.PromiseState === 'rejected') {
                setTimeout(() => {
                    callback(onRejected);
                })
            }

            // 针对异步任务，保存回调函数，待状态发生变化的时候去执行
            if (this.PromiseState === 'pending') {
                // 针对给该Promise实例调用多个then的情况，将其放在数组里面，之后状态改变的时候，逐个去执行
                this.callbacks.push({
                    onResolved: function () {
                        callback(onResolved);
                    },
                    onRejected: function () {
                        callback(onRejected);
                    }
                })
            }
        })
    }

    // 添加catch方法
    catch(onRejected) {
        return this.then(undefined, onRejected);
    }

    // 添加resolve方法
    static resolve(type) {
        return new Promise((resolve, reject) => {
            // Promise实例对象，则根据实例对象状态决定
            if (type instanceof Promise) {
                type.then(value => {
                    resolve(value);
                }, reason => {
                    reject(reason);
                })
            } else {
                // 非Promise实例对象，则返回成功的Promise实例对象
                resolve(type);
            }
        })
    }

    // 添加reject方法
    static reject(value) {
        return new Promise((resolve, reject) => {
            reject(value);
        });
    }

    // 添加all方法
    static all(promises) {
        if (!Array.isArray(promises)) {
            // 如果不是数组，则抛出异常
            throw new Error(`${typeof promises} is not iterable (cannot read property Symbol(Symbol.iterator))`);
        }

        return new Promise((resolve, reject) => {

            // 声明一个标识为，表示实例成功的个数，当前仅当count === promises.length的时候，才会将返回的Promise实例对象状态改为成功
            // 只要promises中有一个对象的状态改为失败，则立即改变当前的Promise实例对象为失败
            let count = 0;
            // 保存成功实例对象的返回的结果
            let arr = [];
            for (let i = 0; i < promises.length; i++) {

                // 每个Promise实例都有一个then方法，可以通过then方法获取到当前实例的状态变化
                promises[i].then(value => {
                    // 当前实例变为成功，必然会走这
                    // 只要成功，则增加1 表明成功的个数
                    count++;
                    // 保存成功实例对象的返回结果，注意顺序问题
                    arr[i] = value;
                    if (count === promises.length) {
                        // 只要实例对象全部都成功，才改变实例对象状态，且把保存的所有实例结果的数组作为当且实例对象的值传入
                        resolve(arr);
                    }
                }, reason => {
                    // 只要失败，直接改变当且对象的状态为失败
                    reject(reason);
                })
            }
        })
    }

    // 添加race方法
    static race(promises) {
        if (!Array.isArray(promises)) {
            // 如果不是数组，则抛出异常
            throw new Error(`${typeof promises} is not iterable (cannot read property Symbol(Symbol.iterator))`);
        }

        return new Promise((resolve, reject) => {
            // 只要实例对象中有一个状态发生改变，则直接改变当前实例对象的状态
            promises.forEach(item => {
                item.then(value => {
                    resolve(value);
                }, reason => {
                    reject(reason);
                })
            })
        })
    }
}
```













































































































































































































