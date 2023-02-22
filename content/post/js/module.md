---
layout: post
title: "JS体系之一【模块化】"
subtitle: ""
date: 2022-08-24 09:56:34
author: "汪洋龙"
categories: []
image: "img/coffee-bg.jpg"
description: "争来争去，还是得统一"
---

## 导读

[前端模块化开发那点历史](https://github.com/seajs/seajs/issues/588) 👈


## IIFE

1. 定义

IIFE: `Immediately Invoked Function Expression`，意为 `立即调用的函数表达式`

2. 为什么需要 `IIFE`


## CommonJS

## AMD

## CMD

## UMD

## ESM

```js
// counter.mjs
let counter = 10;
export default counter;

// index.mjs
import myCounter from './counter.mjs';
myCounter += 1;
console.log(myCounter);
```

> 解析：引入的模块是只读的，不可修改

```
index.mjs:2 Uncaught TypeError: Assignment to constant variable.
    at index.mjs:2:11
```


---



```js
// index.js
console.log('running index.js');
import { sum } from './sum.js';
console.log(sum(1, 2));

// sum.js
console.log('running sum.js');
export const sum = (a, b) => a + b;
```

```
running sum.js
running index.js
3
```

> import 命令是编译阶段执行，所以代码运行之前被导入的模块会先运行。

```js
// index.js
console.log('running index.js');
const { sum } = require('./a.js');
console.log(sum(1, 2));

// sum.js
console.log('running sum.js');
exports.sum = (a, b) => a + b;
```

```
running index.js
running sum.js
3
```


