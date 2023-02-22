---
layout: post
title: "JavaScript modules(译文)"
subtitle: "一种JS新的模块化"
date: 2022-08-23 09:56:34
author: "汪洋龙"
categories: []
image: "img/coffee-bg.jpg"
description: "JS modules"
tags: 
    - "译文"
---

[原文地址：JavaScript modules](https://v8.dev/features/modules) 👈


# JavaScript modules

JavaScript modules 现在[支持所有主流浏览器](https://caniuse.com/es6-module) 👈。

本文解释了如何使用 `JS modules`，如何正确部署，`Chrome` 团队在未来如何让 `modules` 更好。

## 什么是 JS modules?

`JS modules`（也被熟知为 `ES modules` 或者 `ECMAScript modules`）是一个重要的新特性，或者说是新特性的集合。过去你可能用过 `userland` JavaScript 模块系统。可能[在 `Node.js` 中用过 CommonJS](https://nodejs.org/docs/latest-v10.x/api/modules.html) 👈，或者是 [`AMD`](https://github.com/amdjs/amdjs-api/blob/master/AMD.md) 👈，或者别的。所有这些模块系统都有一个共同点：允许你导入导出。

`JavaScript` 现在已经为此提供了标准化的语法。在一个模块里，你可以用关键词 `export` 去导出任何东西。一个 `const`，一个 `function`，或者是其他绑定变量或者声明。只需要在变量语句或者声明前加上 `export` 就行了：

```js
// 📁 lib.mjs
export const repeat = (string) => `${string} ${string}`;
export function shout(string) {
    return `${string.toUpperCase()}!`;
}
```

然后用关键词 `import` 从另一个模块导入该模块。我们从 `lib` 模块导入了 `repeat` 和 `shout`，并在 `main` 模块中使用。

```js
// 📁 main.mjs
import {repeat, shout} from './lib.mjs';
repeat('hello');
// → 'hello hello'
shout('Modules in action');
// → 'MODULES IN ACTION!'
```

你也可以从一个模块导出*默认值*：

```js
// 📁 lib.mjs
export default function(string) {
  return `${string.toUpperCase()}!`;
}
```

这种用*默认值*导入可以在导入的时候随便起名：

```js
// 📁 main.mjs
import shout from './lib.mjs';
//     ^^^^^
```

`modules` 与 `classic` 脚本有一些不同：

- `modules` 默认开启 [`严格模式`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) 👈
- `HTML-style comment syntax is not supported in modules, although it works in classic scripts.`  `译者注：这句有点问题`

```js
// 在 JavaScript 中不要使用 HTML 风格的注释
const x = 42; <!-- TODO: Rename x to y.
// 使用单行注释：
const x = 42; // TODO: Rename x to y.
```

- `modules` 存在顶级词法作用域。意味着对于 `var foo = 42;` 来说，在 `module` 不会创建名为 `foo` 的全局变量，可以通过浏览器访问 `window.foo`，但 `classic` 脚本可以
- 同样的，`modules` 中使用 `this` 并不指向全局 `this`，而是 `undefined`。（如果想要访问全局 `this` 用 [`globalThis`](https://v8.dev/features/globalthis) 👈）
- 新的 `import` 和 `export` 语法只能用于 `modules` —— `classic` 脚本并不生效
- [`Top-level await`](https://v8.dev/features/top-level-await) 👈 在 `modules` 中可用，而 `classic` 脚本不可用。

正因为这些不同，*把同样的JavaScript代码当成 `module` 或者 `classic` 脚本，表现也不尽相同*。所以 `JavaScript` 运行时需要知道哪个是 `modules`。


## 在浏览器中使用 JS modules

在网页中，我们可以通过在 `script` 元素上设置 `type` 为 `module` 来告诉浏览器使用 `module`。

```html
<script type="module" src="main.mjs"></script>
<script nomodule src="fallback.js"></script>
```

能识别 `type="module"` 的浏览器会忽略标记了 `nomodule` 属性的脚本文件。这意味着你可以在支持 `module` 的浏览器中提供以 `module` 形式的 `main.mjs`，而其他浏览器，可以用 `fallback.js`。能够区分开这些的能力是很棒的，就算仅仅是为了性能！试想看：只有现代浏览器支持 `modules`。如果一个浏览器可以理解你的 `module` 代码，那它也支持 [`modules` 之前的特性](https://codepen.io/samthor/pen/MmvdOM) 👈，比如箭头函数或者 `async-await`。你不再需要在模块包中转译这些新特性！你可以[为现代浏览器提供更小且大部分未编译的基于模块的有效负载](https://philipwalton.com/articles/deploying-es2015-code-in-production-today/) 👈。只有旧版浏览器才能获得 `nomodule` 有效负载。

因为[`modules` 默认是 `deferred`](https://v8.dev/features/modules#defer) 👈，你可能也想以 `deferred` 的方式加载 `nomodule` 脚本。

```html
<script type="module" src="main.mjs"></script>
<script nomodule defer src="fallback.js"></script>
```

### 针对于浏览器在 modules 和 classic 脚本之间的不同

现在你知道了，`modules` 和 `classic` 脚本是不一样的。除了我们上面概述的与平台无关的差异之外，还有一些特定于浏览器的差异。

比如说，`modules` 只会加载一次，`classic` 脚本则是你添加多少次，它加载多少次。

```html
<script src="classic.js"></script>
<script src="classic.js"></script>
<!-- classic.js 执行多次 -->

<script type="module" src="module.mjs"></script>
<script type="module" src="module.mjs"></script>
<script type="module">import './module.mjs';</script>
<!-- module.mjs只会加载一次 -->
```

此外，`module` 脚本及其依赖项是用 `CORS` 获取的。这意味着任何跨域的 `module` 脚本必须提供正确的头信息，比如 `Access-Control-Allow-Origin: *`。对于经典脚本来说是不适用的。

另外的一个不同是关于 `async` ，它下载的脚本不会阻塞 `HTML` 渲染（类似于 `defer`），除此之外尽可能去执行，不保证顺序，不等待 `HTML` 渲染的完成。 `async` 属性对内联 `classic` 脚本不起作用，但对于内联 `<script type="module">` 是生效的。

### 关于文件后缀的说明

你可能注意到了我们用 `.mjs` 的文件后缀表明 `modules`。在网页中，文件后缀并不重要，只要它符合 [`JavaScript MIME type text/javascript`](https://html.spec.whatwg.org/multipage/scripting.html#scriptingLanguages:javascript-mime-type) 👈。浏览器知道它是 `module`，因为脚本标签上有 `type` 属性。

尽管如此，我们还是建议对 `module` 使用 `.mjs` 扩展名，原因有 `2` 个：

1. 在开发阶段，`.mjs` 后缀可以让你和查看你项目的其他人非常清楚当前文件是一个 `module`，而不是 `classic` 脚本。（只看代码，并不总是可以分辨出来）就像上面提到的，`module` 的处理方式与 `classic` 脚本不同，因此差异非常重要！

2. 它可以确保你的文件可以被诸如 [`Node.js`](https://nodejs.org/api/esm.html#esm_enabling) 👈  和 [`d8`](https://v8.dev/docs/d8) 👈 这类的运行时、`Babel` 这类的构建工具解析为 `module`。虽然这些环境和工具都具有通过配置将具有其他扩展名的文件解释为 `module` 的专有方式，但 `.mjs` 扩展名是确保文件被视为模块的交叉兼容方式。

> Note：在 `Web` 部署 `.mjs`， your web server needs to be configured to serve files with this extension using the appropriate Content-Type: text/javascript header, as mentioned above。另外，你可能想配置一下编辑器，可以让 `.mjs` 和 `.js` 文件一样获得语法高亮。大多数现代的编译器已经默认这么做了。


### Module 说明符

当 `import` 模块时，指定模块位置的字符串被称为 “模块说明符” 或 “导入说明符”。在之前的例子，模块说明符是 `'./lib.mjs'`

```js
import {shout} from './lib.mjs';
//                  ^^^^^^^^^^^
```

在浏览器中用模块说明符会有一些限制🚫。目前不支持所谓的“裸”模块说明符。[指定](https://html.spec.whatwg.org/multipage/webappapis.html#resolve-a-module-specifier) 👈 当前限制是为了在将来，浏览器可以像如下所示，允许自定义模块加载器给“裸”模块说明符赋予特殊含义。

```js
// 目前还不支持
import {shout} from 'jquery';
import {shout} from 'lib.mjs';
import {shout} from 'modules/lib.mjs';
```

另一方面，下面的例子是支持的：

```js
// 支持
import {shout} from './lib.mjs';
import {shout} from '../lib.mjs';
import {shout} from '/modules/lib.mjs';
import {shout} from 'https://simple.example/modules/lib.mjs';
```

目前而言，模块加载器必须是完整路径，或者开头是 `/`、`./` 或者 `../.` 的相对路径。

### Modules 默认是 deferred
 
`classic` `<script>` 默认会阻塞 `HTML` 渲染。你可以加上 [`defer`](https://html.spec.whatwg.org/multipage/scripting.html#attr-script-defer) 👈 属性，确保脚本的下载和 `HTML` 的解析是并行的。

![async-defer](https://v8.dev/_img/modules/async-defer.svg)


`module` 脚本默认 `defer`。因此在 `<script type="module">` 标签中不用添加 `defer`！不仅主要的 `module` 下载和 `HTML` 的解析是平行的，还包括所有依赖的模块！

## 其他 module 特性

### 动态 `import()`

到目前为止，我们只用了静态 `import`。静态 `import` 就是需要整个模块图下载并执行，然后主代码才能运行。有时候，我们并不想要预先加载模块，而是需要的时候加载，仅仅是在需要的时候 —— 比如说，就是用户点击一个链接或者按钮的时候。这就提高了初始化时候的加载性能。[动态 `import()`](https://v8.dev/features/dynamic-import) 👈 让这变成了可能！

```html
<script type="module">
  (async () => {
    const moduleSpecifier = './lib.mjs';
    const {repeat, shout} = await import(moduleSpecifier);
    repeat('hello');
    // → 'hello hello'
    shout('Dynamic import in action');
    // → 'DYNAMIC IMPORT IN ACTION!'
  })();
</script>
```

与静态 `import` 不同，动态 `import()` 可以在常规脚本中使用。在你目前的代码库中，开始逐步用 `module` 就会变得很简单。查看更多细节，可以参阅[我们关于动态 `import()` 的文章](https://v8.dev/features/dynamic-import) 👈。

> 注释：[`webpack` 有它自己的 `import()` 版本](https://web.dev/use-long-term-caching/)，它巧妙的把导入的模块从它的主包分离出来，然后拆分为自己的块。

`import.meta`

另一个 `module` 相关的新特性是 `import.meta`，它提供了当前模块的元数据。你得到确切的元数据并不是 `ECMAScript` 的一部分；它依赖宿主环境。比如说，在浏览器中你会得到与 `Node.js` 环境不同的元数据。

这是个web端 `import.meta` 的例子。默认情况下，图片在 `HTML` 中是相对当前路径加载的。`import.meta.url` 让加载相对于当前模块的图片成为可能。


```js
function loadThumbnail(relativePath) {
  const url = new URL(relativePath, import.meta.url);
  const image = new Image();
  image.src = url;
  return image;
}

const thumbnail = loadThumbnail('../img/thumbnail.png');
container.append(thumbnail);
```

## 性能建议

### 继续打包

有了 `modules`，开发网站时，就可以不需要诸如 `webpack`、`Rollup`、`Parcel` 的打包工具了。下面场景直接用原生 `JS` 模块就很好：

- 本地开发过程中
- 生产环境中，一共不超过 100 个模块的并且依赖树相对较浅（即最大层级深度不超过5）的小型web应用

尽管如此，当我们了解了[当加载约300个模块组成的模块库时，Chrome加载管道的瓶颈分析](https://docs.google.com/document/d/1ovo4PurT_1K4WFwN2MYmmgbLcr7v6DRQN67ESVA-wq0/pub)，打包应用的性能是优于未打包的。

![renderer-main-thread-time-breakdown](https://v8.dev/_img/modules/renderer-main-thread-time-breakdown.svg)

其中一个原因就是，静态 `import/export` 语法是可以静态分析的，所以它可以帮助打包工具通过消除未使用的导出优化你的代码。静态 `import` 和 `export` 就不仅仅是语法了，它们是一个关键的工具特性！

*我们通常建议，在生产环境中部署模块还是继续打包*。某种程度上，打包是一个类似缩小代码的优化：它可以带来性能优势，因为最终交付更少的代码。打包有同样的效果！继续打包！

像往常一样，[DevTools的代码覆盖功能](https://developer.chrome.com/blog/#coverage)👈 可以帮你分辨出给用户推送了哪些不必要的代码。我们同时建议用[代码分割](https://web.dev/use-long-term-caching/#lazy-loading)👈 去拆包并延迟加载non-First-Meaningful-Paint 关键脚本。

### 打包的取舍 vs. 运送未打包模块

像往常的web开发一样，任何事情都是一种取舍。运送未打包的模块可能会降低初始化加载的性能（冷缓存），但对于后续的访问（热缓存），与运送一个没有代码拆分的单个包相比，实际上提高了性能。对于一个 200 KB 的代码库，更改单个细粒度模块并将其作为后续访问的唯一从服务器获取比重新获取整个包要好得多。

如果你更关心具有热缓存的访问者的体验，而不是首次访问性能，并且网站的细粒度模块少于几百个，你可以尝试发布未打包模块，衡量冷负载和热负载的性能影响，然后做出数据驱动决策！

浏览器工程师们正努力提升开箱即用模块的性能。随着时间推移，我们希望可以在更多场景下，让发布未打包的模块成为可行。

### 使用细粒度的模块

养成使用小的、细粒度的模块编写代码的习惯。在开发过程中，通常每个模块只有几个导出比手动将大量导出合并到一个文件中要好。

考虑一个名为 `./util.mjs` 的模块名导出了 `drop`，`pluck`，`zip`这三个函数：

```js
export function drop() { /* … */ }
export function pluck() { /* … */ }
export function zip() { /* … */ }
```

如果你的代码库只需要 `pluck` 函数，你可能会这么导入：

```js
import {pluck} from './util.mjs';
```

这种情况下，（没有打包步骤）即使你只需要导出一个，浏览器仍然会下载，解析，编译整个 `./util.mjs` 模块。这就很浪费！

如果 `pluck` 并不和 `drop` 和 `zip` 共享任何代码，最好把它移动到自己的细粒度模块中，比如 `./pluck.mjs`。

```js
export function pluck() { /* … */ }
```

然后我们可以导入 `pluck` 而不需要处理 `drop` 和 `zip` 的开销： 

```js
import {pluck} from './pluck.mjs';
```

> 注释：可以根据个人习惯，使用 `default` 导出而不是命名导出。

这不仅能让你的代码保持美观和简单，it also reduces the need for dead-code elimination as performed by bundlers. 如果代码中其中一个模块没有用，那它就不会导入，也就是说浏览器不会下载它。这么用的模块可以单独在浏览器中实现[代码缓存](https://v8.dev/blog/code-caching-for-devs) 👈（实现这个的基础已经登录V8，并且正努力在chrome中启用它。）

用小型，细颗粒度的模块可以帮你在未来可以使用[原生打包解决方案](https://v8.dev/features/modules#web-packaging) 👈 作准备。


### 模块预加载

你可以进一步用[<link rel="modulepreload">](https://developer.chrome.com/blog/modulepreload/) 👈 来优化模块的交付。这可以让浏览器预加载甚至预解析和预编译模块及其依赖项。

```js
<link rel="modulepreload" href="lib.mjs">
<link rel="modulepreload" href="main.mjs">
<script type="module" src="main.mjs"></script>
<script nomodule src="fallback.js"></script>
```

这对于较大的依赖树尤其重要。如果没有 `rel="modulepreload"`，浏览器需要执行多个 HTTP 请求才能找出完整的依赖关系树。但是，如果用 `rel="modulepreload"` 声明依赖模块脚本的完整列表，浏览器就不用逐步发现这些依赖关系。

### 使用 HTTP/2 协议

就它的[多路复用](https://web.dev/performance-http2/#request_and_response_multiplexing)而言，尽可能用 `HTTP/2` 一直是很好的性能建议。通过 HTTP/2 多路复用，可以同时传输多个请求和响应消息，这有利于加载模块树。

Chrome 团队调查了另一个 HTTP/2 功能，特别是 [HTTP/2 服务器推送](https://web.dev/performance-http2/#server_push) 👈，可能是成为部署高度模块化应用程序的实用解决方案。不爽的是，[HTTP/2 服务器推送很难正确处理](https://jakearchibald.com/2017/h2-push-tougher-than-i-thought/)，Web 服务器和浏览器的实现目前尚未针对高度模块化的 Web 应用程序用例进行优化。比如说，仅推送用户尚未缓存的资源是很困难的，并且通过将源的整个缓存状态传达给服务器来解决这个问题存在隐私风险。

所以无论如何，继续使用 HTTP/2！记住，HTTP/2 服务器推送（很不幸）不是灵丹妙药。

## Web 采用 JS 模块化

JS 模块化逐步在 Web 端采用。[我们的用量统计](https://chromestatus.com/metrics/feature/timeline/popularity/2062) 👈 显示目前 0.08% 的页面采用了 `<script type="module">`。注意这个数字并不包含比如动态 `import()` 或者 [worklets](https://html.spec.whatwg.org/multipage/worklets.html)其他入口。

## JS 模块化的未来

Chrome 团队正致力于以各种方式改善 JS 模块的开发时体验。我们来说说。

### 更快和确定性的模块解析算法

我们提议对模块解析算法进行更改，以解决速度和确定性方面的缺陷。新算法现在同时存在于 ][HTML 规范](https://github.com/whatwg/html/pull/2991) 👈 和 [ECMAScript 规范](https://github.com/tc39/ecma262/pull/1006) 👈 中，并在 [Chrome 63](https://bugs.chromium.org/p/chromium/issues/detail?id=763597) 👈 中实现。预计这种改进很快就会出现在更多浏览器中！

新算法效率更高，速度更快。在依赖图的大小上，旧算法的计算复杂度是二次方的，即𝒪(n²)，Chrome 的实现也是如此。新算法是线性的，即𝒪(n)。

此外，新算法以确定性的方式报告解析错误。给定一个包含多个错误的图，旧算法的不同运行可能会报告不同的错误，这些错误是导致解决失败的原因。这样就徒增调试难度。新算法保证每次都报告相同的错误。

### Worklets 和 web workers

Chrome 现在实现了[worklets](https://html.spec.whatwg.org/multipage/worklets.html) 👈 ，可以让web开发者在web浏览器的“低级部分”自定义硬编码逻辑。通过worklets，web开发者可以把JS模块喂给渲染管道或者音频处理管道（未来可能会有更多的管道！）

Chrome 65 支持 [PaintWorklet](https://developer.chrome.com/blog/paintapi/) 👈（又名 CSS Paint API）来控制 DOM 元素的绘制方式。

```js
const result = await css.paintWorklet.addModule('paint-worklet.mjs');
```

Chrome 66 支持 [AudioWorklet](https://developer.chrome.com/blog/audio-worklet/) 👈，它允许您使用自己的代码控制音频处理。

相同的 Chrome 版本为 AnimationWorklet 启动了[OriginTrial](https://groups.google.com/a/chromium.org/g/blink-dev/c/AZ-PYPMS7EA/m/DEqbe2u5BQAJ) 👈，它可以创建滚动链接和其他高性能程序动画。

最后，[LayoutWorklet](https://drafts.css-houdini.org/css-layout-api/)（又名 CSS Layout API）现在在 Chrome 67 中实现

我们正在努力增加对在 Chrome 中使用 JS 模块和专用网络工作者的支持。您已经可以在启用 `chrome://flags/#enable-experimental-web-platform-features` 的情况下尝试此功能。

```js
const worker = new Worker('worker.mjs', { type: 'module' });
```

JS 模块对共享工作者和服务工作者的支持即将推出：

```js
const worker = new SharedWorker('worker.mjs', { type: 'module' });
const registration = await navigator.serviceWorker.register('worker.mjs', { type: 'module' });
```

### 导入映射

在 `Node.js/npm` 中，通常用“包名”导入JS模块。例如：

```js
import moment from 'moment';
import {pluck} from 'lodash-es';
```

目前，[根据 HTML 规范](https://html.spec.whatwg.org/multipage/webappapis.html#resolve-a-module-specifier) 👈，此类“裸导入说明符”会引发异常。我们的导入地图提案允许此类代码在网络上运行，包括在生产应用程序中。导入映射是一种 JSON 资源，可帮助浏览器将裸导入说明符转换为完整的 URL。

导入映射仍处于提案阶段。尽管我们对它们如何处理各种用例进行了很多思考，但我们仍在与社区互动，还没有编写完整的规范。欢迎反馈！

### Web打包: 原生 bundles

Chrome 加载团队目前正在探索一种原生 Web 打包格式，作为一种分发 Web 应用程序的新方式。网络打包的核心特点是：

签名的 HTTP 交换允许浏览器相信单个 HTTP 请求/响应对是由它声称的来源生成的；捆绑的 HTTP 交换器，即交换器的集合，每个交换器都可以签名或未签名，并带有一些描述如何将包作为一个整体解释的元数据。

结合起来，这样的 Web 打包格式将使多个同源资源能够安全地嵌入到单个 HTTP GET 响应中。

现有的捆绑工具（例如 webpack、Rollup 或 Parcel）当前会发出单个 JavaScript 捆绑包，其中丢失了原始单独模块和资产的语义。使用本机捆绑包，浏览器可以将资源解绑回其原始形式。简而言之，您可以将捆绑的 HTTP 交换想象为可以通过目录（清单）以任何顺序访问的资源包，并且可以根据其相对重要性有效地存储和标记所包含的资源，同时保持单个文件的概念。因此，本机捆绑包可以改善调试体验。在 DevTools 中查看资产时，浏览器可以精确定位原始模块，而无需复杂的源映射。

原生包格式的透明性开启了各种优化机会。例如，如果浏览器已经在本地缓存了本地包的一部分，它可以将其传送到 Web 服务器，然后只下载丢失的部分。

Chrome 已经支持该提案的一部分（SignedExchanges），但捆绑格式本身及其在高度模块化应用程序中的应用仍处于探索阶段。非常欢迎您在存储库中提供反馈或通过电子邮件发送至 loading-dev@chromium.org！

### 分层的 API

发布新功能和 Web API 会产生持续的维护和运行时成本——每个新功能都会污染浏览器命名空间，增加启动成本，并代表在整个代码库中引入错误的新表面。分层 API 旨在以更具可扩展性的方式在 Web 浏览器中实现和发布更高级别的 API。 JS 模块是分层 API 的关键支持技术：

- 由于模块是显式导入的，因此要求通过模块公开分层 API 可确保开发人员只需为他们使用的分层 API 付费。
- 因为模块加载是可配置的，所以分层 API 可以有一个内置机制，用于在不支持分层 API 的浏览器中自动加载 polyfill。

模块和分层 API 如何协同工作的细节仍在制定中，但目前的提案看起来像这样：

```html
<script
  type="module"
  src="std:virtual-scroller|https://example.com/virtual-scroller.mjs"
></script>
```

`<script>` 元素从浏览器的内置分层 API 集 (std:virtual-scroller) 或从指向 polyfill 的后备 URL 加载虚拟滚动 API。这个 API 可以做任何 JS 模块在 Web 浏览器中可以做的事情。一个示例是定义自定义 `<virtual-scroller>` 元素，以便根据需要逐步增强以下 HTML：

```html
<virtual-scroller>
  <!-- 内容放在这 -->
</virtual-scroller>
```



# 总结【译者注】

1. 用 import export 实现导入导出
2. 正常
3. ESM 默认开启严格模式
4. 
