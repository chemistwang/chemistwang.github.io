---
layout: post
title: "JS体系之四【事件循环】"
subtitle: ""
date: 2022-08-31
author: "汪洋龙"
categories: []
image: "img/coffee-bg.jpg"
description: "很重要"
---

# 事件循环

推荐下面这个视频，讲的非常好。

[JavaScript中的事件循环介绍 -Jake Archibald【Bilibili】](https://www.bilibili.com/video/BV1XQ4y1P7L7?from=search&seid=4063426400438383115) 👍

[JavaScript中的事件循环介绍 -Jake Archibald【YouTube】](https://www.youtube.com/watch?v=cCOL7MC4Pl0&t=166s&ab_channel=JSConf) 👈


视频最后有一个例子：

```js
let btn = document.getElementById('btn')
btn.addEventListener('click', () => {
    Promise.resolve().then(() => console.log('m1'))
    console.log('listen 1')
})

btn.addEventListener('click', () => {
    Promise.resolve().then(() => console.log('m2'))
    console.log('listen 2')
})

// btn.click()
```

```js
let btn = document.getElementById('btn')
btn.addEventListener('click', () => {
    Promise.resolve().then(() => console.log('m1'))
    console.log('listen 1')
})

btn.addEventListener('click', () => {
    Promise.resolve().then(() => console.log('m2'))
    console.log('listen 2')
})

btn.click()
```

不妨先试试，看看输出的具体顺序，解析在 `练习10`。


## 前言

- **单线程**：这个概念很重要，它决定了 `JS` 的很多行为
- **任务队列**：`Task Queues`，既然是单线程，浏览器的特殊性决定了遇到阻塞性事件不能等待⌛️。怎么办呢，不妨提交一个任务，稍后处理

## 运行机制

1. 同步任务都在主线程执行，形成一个执行栈（`execution context stack`）
2. 对于异步任务，主线程之外，存在任务队列（`task queue`）
3. 任务队列分为两大类，分别是宏任务（`macro task`）和微任务（`micro task`）
4. 只有同步任务全部完成，即执行栈清空，才会执行异步任务
5. 清空完当前微任务，才会再去执行下一个宏任务，如下代码

``` js
for (macroTask of macroTaskQueue) {
    // 1. Handle current MACRO-TASK
    handleMacroTask();
      
    // 2. Handle all MICRO-TASK
    for (microTask of microTaskQueue) {
        handleMicroTask(microTask);
    }
}
```


## 名词解释

### 宏任务 (macro task)

- `XHR` 回调
- 事件回调（鼠标键盘事件）
- `setImmediate`
- `setTimeout`
- `setInterval`
- indexedDB 数据库操作等 I/O

**说明**：`setTimeout` 的延时参数始终 `相对于主程序执行完毕的时间`，并且多个 `setTimeout` 执行的先后顺序也是由这个延迟时间决定


### 微任务 (micro task)

- `process.nextTick`
- `Promise.then`
- `MutationObserver`


## 练习

### 练习1

```js
// 代码输出
console.log("global");

setTimeout(function() {
  console.log("timeout1");
  new Promise(function(resolve) {
    console.log("timeout1_promise");
    resolve();
  }).then(function() {
    console.log("timeout1_then");
  });
}, 2000);

for (var i = 1; i <= 5; i++) {
  setTimeout(function() {
    console.log(i);
  }, i * 1000);
  console.log(i);
}

new Promise(function(resolve) {
  console.log("promise1");
  resolve();
}).then(function() {
  console.log("then1");
});

setTimeout(function() {
  console.log("timeout2");
  new Promise(function(resolve) {
    console.log("timeout2_promise");
    resolve();
  }).then(function() {
    console.log("timeout2_then");
  });
}, 1000);

new Promise(function(resolve) {
  console.log("promise2");
  resolve();
}).then(function() {
  console.log("then2");
});
```

``` js
// global
// 1
// 2
// 3
// 4
// 5
// promise1
// promise2
// then1
// then2
// 6
// timeout2
// timeout2_promise
// timeout2_then
// timeout1
// timeout1_promise
// timeout1_then
// 6
// 6
// 6
// 6
```

**解析**：`Promise`的异步体现在`then`和`catch`中，而里面的代码是同步执行的，所以

第`1`个循环：
- global
- 挂起 `macroTask` setTimeout(...,2000)
- 挂起 `macroTask` setTimeout(...,1000)
- 1
- 挂起 `macroTask` setTimeout(...,2000)
- 2
- 挂起 `macroTask` setTimeout(...,3000)
- 3
- 挂起 `macroTask` setTimeout(...,4000)
- 4
- 挂起 `macroTask` setTimeout(...,5000)
- 5 (这个时候 i 已经变成 6 了)
- promise1
- 挂起 `mircoTask` then1
- 挂起 `macroTask` setTimeout(...,1000)
- promise2
- 挂起 `mircoTask` then2
- `done`
- 发现上述 `2` 个 `microTask`，顺序执行输出 then1、then2

第`2`个循环：
- 发现for循环挂起的setTimeout(...,1000)，输出 6
 
第`3`个循环：
- 发现倒数第二个setTimeout(...,1000)
- timeout2
- timeout2_promise
- 挂起 `microTask` timeout2_then
- `done`
- 发现上述 `1` 个 `microTask`，顺序执行输出 timeout2_then

第`4`个循环：
- 发现第一个setTimeout(...,2000)
- timeout1
- timeout1_promise
- 挂起 `microTask` timeout1_then
- `done`
- 发现上述 `1` 个 `microTask`，顺序执行输出 timeout1_then
 
第`5`个循环
- 发现setTimeout(...,2000)，执行输出 6
- `done`

第`6`个循环
- 发现setTimeout(...,3000)，执行输出 6
- `done`

第`7`个循环
- 发现setTimeout(...,4000)，执行输出 6
- `done`

第`8`个循环
- 发现setTimeout(...,5000)，执行输出 6
- `done`

---

### 练习2

```js
// 代码输出
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}

async function async2() {
  console.log("async2");
}

console.log("script start");

setTimeout(function() {
  console.log("setTimeout");
}, 0);

async1();

console.log("script end");
```

```js
//script start
//async1 start
//async2
//script end
//async1 end
//setTimeout
```

 **解析**：[MDN async 函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function#%E5%8F%82%E8%A7%81) 👈 
 
- await 表达式会暂停整个 async 函数执行进程并让出其控制权
- async 函数一定会返回一个 promise 对象。如果一个 async 函数的返回值看起来不是 promise，那么它将会被隐式地包装在一个 promise 中。

不妨将上述例子改写一下

```js
function async1() {
  console.log("async1 start");
  async2().then(() => {
    console.log("async1 end");
  })
}

function async2() {
  console.log("async2");
  Promise.resolve()
}

console.log("script start");

setTimeout(function() {
  console.log("setTimeout");
}, 0);

async1();

console.log("script end");
```

---

### 练习3

```js
// 代码输出
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}

async function async2() {
  console.log("async2");
}

console.log("script start");

setTimeout(() => {
  console.log("setTimeout");
}, 0);

async1();

new Promise((resolve) => {
  console.log("promise1");
  resolve();
}).then(() => {
  console.log("promise2");
});

console.log("script end");
```

``` js
// script start
// async1 start
// async2
// promise1
// script end
// async1 end
// promise2
// setTimeout
```

---

### 练习4

```js
// 代码输出
setTimeout(() => {
  console.log("setTimeout");
}, 0);

console.log("t1");

fetch("http://localhost:8888")
  .then(function(response) {
    return response.json();
  })
  .then(function(myJson) {
    console.log("myJson");
  })
  .catch(function(err) {
    console.log(err);
  });

console.log("fetch zhi hou");

async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}

async1();

console.log("t2");

new Promise((resolve) => {
  console.log("promise");
  resolve();
}).then(() => {
  console.log("promise.then");
});

console.log("t3");

async function async2() {
  console.log("async2");
}

console.log("t4");
```

``` js
//t1
//fetch zhi hou
//async1 start
//async2
//t2
//promise
//t3
//t4
//async1 end
//promise.then
//setTimeout
//myJson
//error
```

---

### 练习5

```js
// 代码输出
const promise = new Promise((resolve, reject) => {
  console.log(1);
  resolve();
  console.log(2);
});

promise.then(() => {
  console.log(3);
});

console.log(4);
```

```js
//1
//2
//4
//3
```

---

### 练习6

```js
// 代码输出
console.log("script start");

setTimeout(() => {
  console.log("setTimeout...");
}, 1 * 2000);

Promise.resolve()
  .then(function() {
    console.log("promise1");
  })
  .then(function() {
    console.log("promise2");
  });

async function foo() {
  await bar();
  console.log("async1 end");
}

foo();

async function errorFunc() {
  try {
    await Promise.reject("error!!!");
  } catch (e) {
    console.log(e);
  }
  console.log("async1");
  return Promise.resolve("async1 success");
}

errorFunc().then((res) => console.log(res));

function bar() {
  console.log("async2 end");
}

console.log("script end");
```

```js
//script start
//async2 end
//script end
//promise1
//async1 end
//error!!!
//async1
//promise2
//async1 success
//setTimeout...
```

---

### 练习7

```js
// 代码输出
console.log(1);

setTimeout(() => {
  console.log(2);
  Promise.resolve().then(() => {
    console.log(3);
  });
  new Promise((resolve) => {
    console.log(4);
    resolve();
  }).then(() => {
    console.log(5);
  });
});

Promise.reject().then(
  () => {
    console.log(131);
  },
  () => {
    console.log(12);
  }
);

new Promise((resolve) => {
  console.log(7);
  resolve();
}).then(() => {
  console.log(8);
});

setTimeout(() => {
  console.log(9);
  Promise.resolve().then(() => {
    console.log(10);
  });
  new Promise((resolve) => {
    console.log(11);
    resolve();
  }).then(() => {
    console.log(121);
  });
});
```

```js
// 1
// 7
// 12
// 8
// 2
// 4
// 3
// 5
// 9
// 11
// 10
// 121
```

**解析**：[MDN Promise.prototype.then()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) 👈

- then() 方法返回一个 Promise。它最多有 2 个参数：Promise的成功和失败情况的回调函数。

---

### 练习8

```js
// 代码输出
new Promise((resolve, reject) => {
    console.log(1);
    resolve();
})
.then(() => {
    console.log(2);
    new Promise((resolve, reject) => {
        console.log(3);
        setTimeout(() => {
            reject();
        }, 3 * 1000);
        resolve();
    })
    .then(() => {
        console.log(4);
        new Promise((resolve, reject) => {
            console.log(5);
            resolve();
        })
        .then(() => {
            console.log(7);
        })
        .then(() => {
            console.log(9);
        });
    })
    .then(() => {
        console.log(8);
    });
})
.then(() => {
    console.log(6);
});
```

```js
// 1
// 2
// 3
// 4
// 5
// 6
// 7
// 8 
// 9
```

---



### 练习9

```js
// 代码输出
setTimeout(function() {
  console.log(1);
}, 0);
new Promise(function(a, b) {
  console.log(2);
  for (var i = 0; i < 1000; i++) {
    i == 999 && a();
  }
  console.log(3);
}).then(function() {
  console.log(4);
});

console.log(5);
```

``` js
// 2 
// 3
// 5
// 4
// 1
```


### 练习10

```js
let btn = document.getElementById('btn')
btn.addEventListener('click', () => {
    Promise.resolve().then(() => console.log('m1'))
    console.log('listen 1')
})

btn.addEventListener('click', () => {
    Promise.resolve().then(() => console.log('m2'))
    console.log('listen 2')
})

// btn.click()
```

``` js
// listen1
// m1
// listen2
// m2
```

**解析**：点击事件开启了下一个循环，所以需要把当前任务执行完毕。

```js
let btn = document.getElementById('btn')
btn.addEventListener('click', () => {
    Promise.resolve().then(() => console.log('m1'))
    console.log('listen 1')
})

btn.addEventListener('click', () => {
    Promise.resolve().then(() => console.log('m2'))
    console.log('listen 2')
})

btn.click()
```

``` js
// listen1
// listen2
// m1
// m2
```

**解析**：因为是直接触发，所以两个函数处在同一个事件循环中。