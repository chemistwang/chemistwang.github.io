---
layout: post
title: "JS体系之五【this】"
subtitle: ""
date: 2022-09-01
author: "汪洋龙"
categories: []
image: "img/coffee-bg.jpg"
description: "优雅的传递上下文，哼"
---

> [再来 40 道 this 面试题酸爽继续(1.2w 字用手整理)](https://juejin.cn/post/6844904083707396109) 🔥 来呗，先刷刷题

## 绑定规则

`this 的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式`

## 常规情况

### 默认绑定 (优先级最低)

#### 1. 全局对象默认绑定 `window`

```js
function foo() {
  console.log(this.a);
}
var a = 2;
foo(); //2 -> 调用位置：直接调用，只能使用默认绑定
```

#### 2. `'use strict'` 全局对象无法默认绑定

```js
"use strict";
function foo() {
  console.log(this); //undefined 函数内部undefined
  console.log(this.a); //Uncaught TypeError: Cannot read property 'a' of undefined
}
console.log(this); //window
var a = 2;
foo();
```

### 隐式绑定

是否有上下文

缺点：容易造成 `this` 丢失绑定对象

```js
// Q: 代码输出
var length = 10;
function fn() {
  console.log(this.length); //?
}
var obj = {
  length: 5,
  method: function (fn) {
    fn();
    arguments[0]();
  },
};
obj.method(fn);
```

```js
//10
//1
```

---

### 显式绑定

#### 1. 硬绑定

绑定方法：`call`, `apply`, `bind`

```js
// Q: bind , call , apply 区别
```

```js
// 相同点：三者都是绑定 this
// 不同点：bind 只是负责绑定 this 并返回新的方法，并不执行, 绑定之后不再改变

let o = { name: "o" };
let p = { name: "p" };

function show(city) {
  this.name + city;
}

show.bind(o).bind(p)(); //o

//call: 立即执行,参数以字符串形式
show.call(o, "lucus");

//apply: 立即执行,参数以数组形式
show.apply(o, ["lucus"]);
```

---

```js
// Q: call 和 apply 的区别是什么，哪个性能更好一些
```

> apply 转化的是内置的 call，并非 `Function.prototype.call`
>
> apply 最后还是转化成 call 来执行的，call 要更快毫无疑问

#### 2. API 调用的上下文

```js
function foo(el) {
  console.log(el, this.id);
}
var obj = {
  id: "awesome",
};
let list = [1, 2, 3];
list.forEach(foo, obj); //1 awesome 2 awesome 3 awesome
```

### `new` 绑定

第一步： 创建（或者说构造）一个全新的对象。

第二步：这个新对象会被执行 [[原型]] 连接。

第三步：这个新对象会绑定到函数调用的 this。

第四步：如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象。

## 箭头函数

- 箭头函数 `不使用` 上述 `4` 种规则 ！！！⚠️

- 根据外层（函数或者全局）`作用域` 决定 `this`

- 箭头函数的绑定 `无法` 被修改

[箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions) 👈 ， MDN 有这么一句话 `箭头函数...没有自己的this，arguments，super或new.target。...不能用作构造函数。`

## 练习

### 练习 1

```js
// Q: 代码输出
function demo1(x) {
  return arguments.length ? x : "demo";
}

let demo2 = (x) => {
  return arguments.length ? x : "demo";
};

demo1("a");
demo2("a");
```

```js
//1
//Uncaught ReferenceError: arguments is not defined
```

> 解析：`js` 箭头函数是没有 `this` 和 `arguments` 变量的，如果这两个值可以打印，则一定来自父级作用域

### 练习 2

```js
const shape = {
  radius: 10,
  diameter() {
    return this.radius * 2;
  },
  perimeter: () => 2 * Math.PI * this.radius,
};

console.log(shape.diameter());
console.log(shape.perimeter());
```

```js
// 20
// NaN
```

### 练习 3

```js
// 代码输出
var age = 16;
var person = {
  age: 18,
  getAge: function () {
    var age = 22;
    setTimeout(function () {
      alert(this.age);
    }, 1000);
  },
};

person.getAge();
```

```js
// 16
// 解析：若setTimeout的回调为箭头函数。则为18
```

---
