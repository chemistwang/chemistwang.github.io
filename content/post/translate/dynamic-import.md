---
layout: post
title: "Dynamic import( )(译文)"
subtitle: "动态导入"
date: 2022-08-24 14:31:34
author: "汪洋龙"
categories: []
image: "img/coffee-bg.jpg"
description: "Dynamic import"
tags: 
    - "译文"
---

[原文地址：Dynamic import( )](https://v8.dev/features/dynamic-import) 👈

# 动态导入

[动态导入](https://github.com/tc39/proposal-dynamic-import) 👈 引入一种新的类似函数形式的导入方式，和静态导入相比，解锁了新的功能。本文比较两者并概述新功能。

## 静态导入（回顾）

`Chrome 61` 支持[模块](https://v8.dev/features/modules) 👈 中的 ES2015 导入语句。

下面这个模块位于 `./utils.mjs`：

```js
// 默认导出
export default () => {
  console.log('Hi from the default export!');
};

// 导出 `doStuff`
export const doStuff = () => {
  console.log('Doing stuff…');
};
```

这是静态导入使用 `./utils.mjs` 模块的方法：

```html
<script type="module">
  import * as module from './utils.mjs';
  module.default();
  // → logs 'Hi from the default export!'
  module.doStuff();
  // → logs 'Doing stuff…'
</script>
```

> **注释：** 前面的例子用 `.mjs` 扩展名表示它是个 `module`。在 `web` 端，文件的扩展名并不重要，只要文件在 `Content-Type` 请求头中提供了正确的 `MIME` 类型（例如 `JavaScript` 文件的 `text/javascript`）
>
> `.mjs` 在诸如 `Node.js` 和 `d8` 这类的平台也适用。这些平台没有 `MIME` 类型的概念或者其他强制性的约束，就比如 `type="module"` 这种，需要确定是 `module` 还是常规脚本。

这种导入模块的形式是静态的：模块说明符只能是个字符串，并且通过运行前引入绑定到本地作用域。静态导入语法只能在文件顶部使用。

静态导入支持一些很重要的场景，比如说 `static analysis`，`bundling tools` 和`tree-shaking`。

在有些情况下，下面这些就很有用：

- 按需加载模块（或者根据条件加载）
- 运行时计算模块标识符
- 从常规模块导入模块（与 module 相反）

这些静态导入是实现不了的。

## 动态导入 🔥

[动态导入](https://github.com/tc39/proposal-dynamic-import) 👈  就是一种类似函数的导入形式，可以满足这些需求。`import(moduleSpecifier)` 给请求模块的命名空间对象返回一个 `promise`，它是在获取，实例化并且评估自己和所有依赖项之后创建的。

这是动态导入使用 `./utils.mjs` 模块的方法：

``` html
<script type="module">
  const moduleSpecifier = './utils.mjs';
  import(moduleSpecifier)
    .then((module) => {
      module.default();
      // → logs 'Hi from the default export!'
      module.doStuff();
      // → logs 'Doing stuff…'
    });
</script>
```

因为 `import()` 返回的是一个 `promise`，所以可以用 `async/await` 代替 `then` 回调：

```js
<script type="module">
  (async () => {
    const moduleSpecifier = './utils.mjs';
    const module = await import(moduleSpecifier)
    module.default();
    // → logs 'Hi from the default export!'
    module.doStuff();
    // → logs 'Doing stuff…'
  })();
</script>
```

> **注释：** 虽然 `import()` 看上去是一个函数的调用，只是恰好给它一个括号的语法（类似于 [super()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super) 👈 ）。这就是说，`import` 不从 `Function.prototype` 继承，不能用 `call` 或者 `apply`，类似 `const importAlias = import` 这种也不成立 —— 好家伙，`import` 甚至不是一个对象！不过这在实践中并不重要。

下面是在一个单页面应用里，在导航时如何用动态导入实现模块懒加载：


```html
<!DOCTYPE html>
<meta charset="utf-8">
<title>My library</title>
<nav>
  <a href="books.html" data-entry-module="books">Books</a>
  <a href="movies.html" data-entry-module="movies">Movies</a>
  <a href="video-games.html" data-entry-module="video-games">Video Games</a>
</nav>
<main>This is a placeholder for the content that will be loaded on-demand.</main>
<script>
  const main = document.querySelector('main');
  const links = document.querySelectorAll('nav > a');
  for (const link of links) {
    link.addEventListener('click', async (event) => {
      event.preventDefault();
      try {
        const module = await import(`/${link.dataset.entryModule}.mjs`);
        // The module exports a function named `loadPageInto`.
        module.loadPageInto(main);
      } catch (error) {
        main.textContent = error.message;
      }
    });
  }
</script>
```

正确用动态导入实现懒加载是很强大的。出于演示目的，`Addy` 修改了之前用静态导入的一个例子。在升级的版本中，用了动态导入。

> **注释：** 如果你的应用跨域导入了脚本（不管动态还是静态），那么它们需要返回有效的 `CORS` 头信息（比如 `Access-Control-Allow-Origin：*`）。这是因为和常规脚本不同，`module`（和它的一些引入）是通过 `CORS` 获取的。

## 建议

动态和静态导入都很有用。都有各自适合的场景。静态导入适合首屏加载场景下的依赖项。而动态导入就适合按需加载。

