---
layout: post
title: "JS体系之三【Promise】"
subtitle: ""
date: 2022-08-31
author: "汪洋龙"
categories: []
image: "img/coffee-bg.jpg"
description: ""
---

参考资料：

[PromiseA+规范](https://promisesaplus.com/) 👈

## 核心要点

### Promise状态

一个 `promise` 必须是 `3` 种状态之一：`pending`、`fulfilled`、`rejected`

- `pending` 状态，可以转化成 `fulfilled` `rejected` 状态
- `fulfilled` 状态，状态不可变，必须有值，该值不可变
- `rejected` 状态，状态不可变，必须有值，该值不可变

### then方法

一个 `promise` 必须提供 `then` 方法去获得当前值、`fulfilled` 或者 `rejected` 的值。

`then` 方法接受 2 个参数：

```js
promise.then(onFulfilled, onRejected)
```

1. `onFulfilled`、 `onRejected` 均为 `可选` 参数

    - 若 `onFulfilled` 不是函数，必须忽略
    - 若 `onRejected` 不是函数，必须忽略

2. 若 `onFulfilled` 是函数：

    - promise `fulfilled` 状态之后必须调用，promise的值作为该函数第 `1` 个参数
    - promise `fulfilled` 状态之前禁止调用
    - 只能调用一次

3. 若 `onRejected` 是函数：

    - promise `rejected` 状态之后必须调用，promise的值作为该函数第 `1` 个参数
    - promise `rejected` 状态之前禁止调用
    - 只能调用一次

4. 在执行上下文栈只包含平台代码之前，`onFulfilled` 或者 `onRejected` 不能被调用

5. `onFulfilled` 和 `onRejected` 必须作为函数调用（不能有 `this`）

6. 同一个 promise 可以调用多次 `then` 方法

    - 如果 promise `fulfilled` 状态，`onFulfilled` 回调必须按照 `then` 顺序执行
    - 如果 promise `rejected` 状态，`onRejected` 回调必须按照 `then` 顺序执行

7. `then` 方法必须返回一个 `promise`

    - 如果 `onFulfilled` 或者 `onRejected` 返回一个值为 x，执行 `[[Resolve]](promise2, x)`
    - 如果 `onFulfilled` 或者 `onRejected` 抛出错误 e，promise2 必须为 `rejected`
    - 如果 `onFulfilled` 不是函数，并且 promise1 `fulfilled` 状态，promise2 必须 `fulfilled`，并且带有和 promise1 一样的值
    - 如果 `onRejected` 不是函数，并且 promise1 `rejected` 状态， promise2 必须 `rejected`，并且带有和 promise1 一样的错误信息

## 方法

这 `2` 个方法接受一个 `iterable` 类型参数，即 `Array`，`Set`，`Map`

不妨先定义 `4` 个 Promise

```js
let Promise1 = new Promise(function (resolve, reject) {
    setTimeout(() => {
        resolve(1)
    }, 0)
})
let Promise2 = new Promise(function (resolve, reject) {
    setTimeout(() => {
        resolve(2)
    }, 1000)
})
let Promise3 = new Promise(function (resolve, reject) {
    setTimeout(() => {
        resolve(3)
    }, 3000)
})
let Promise4 = new Promise(function (resolve, reject) {
    setTimeout(() => {
        reject(4)
    }, 1500)
})

const promise_set = new Set();
promise_set.add(Promise1);
promise_set.add(Promise2);
promise_set.add(Promise3);

const iterable = [Promise1, Promise2, Promise3] || promise_set
```

### Promise.all

- 所有都返回执行回调

```js
Promise.all([Promise1, Promise2, Promise3])
.then((value) => {
    console.log(value) // [1,2,3] 3000ms后打印
}, (error) => {
    console.log(error)
})

```

- 只要有一个 `reject` 执行立即抛出错误

```js
Promise.all([Promise1, Promise2, Promise4])
.then((value) => {
    console.log(value)
}, (error) => {
    console.log(error) // 4 1500ms后打印
})
```

### Promise.race

那就是看谁跑的快了

```js
Promise.race([Promise1, Promise2])
.then((value) => {
   console.log(value) // 1
}, (error) => {
    console.log(error)
})
```

```js
Promise.race([Promise2, Promise4])
.then((value) => {
   console.log(value) // 2
}, (error) => {
    console.log(error)
})
```

```js
Promise.race([Promise3, Promise4])
.then((value) => {
   console.log(value)
}, (error) => {
    console.log(error) // 4
})
```


## 手写Promise




## 执行逻辑

推荐阅读：[V8 Promise源码全面解读](https://juejin.cn/post/7055202073511460895) 👈

有时候对于多个 `then` 执行顺序，通过源码的指导会明确很多。

## 练习

### 练习1

```js
Promise.resolve()
    .then(() => {
        console.log(0);
        return Promise.resolve(4)
    }).then(res => {
        console.log(res);
    })

Promise.resolve().then(() => {
    console.log(1);
}).then(() => {
    console.log(2);
}).then(() => {
    console.log(3);
}).then(() => {
    console.log(5);
}).then(() => {
    console.log(6);
})
```
