---
layout: post
title: "「React源码之一」 搭建源码调试环境"
subtitle: "「源码阅读」第二层：掌握整体工作流程局部细节"
date: 2022-08-03
author: "汪洋龙"
categories: [React]
image: "img/react-bg.jpg"
tags:
  - "react"
---

**实现目标：在 `VSCode` 中执行断点调试 [`官方Github仓库中`](https://github.com/facebook/react.git) 👈 的 `React原始源码` 并可以定位到具体文件**

> 版本说明：`React18.02`

思路说明：利用源码编译出来的 `sourcemap` 寻找对应的源码文件

## 第一步：构建源码 sourcemap

### 1. 下载源码

```bash
git clone https://github.com/facebook/react.git
```

### 2. `npm i` 安装依赖

> warning 报错

```bash
➜  react git:(main) npm i
npm ERR! code EUNSUPPORTEDPROTOCOL
npm ERR! Unsupported URL Type "link:": link:./scripts/eslint-rules

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/wyl/.npm/_logs/2022-07-22T11_12_55_347Z-debug.log
```

解决方案：修改 `package.json` 的依赖并重新执行命令

```json
...
   // "eslint-plugin-react-internal": "link:./scripts/eslint-rules",
   "eslint-plugin-react-internal": "file:./scripts/eslint-rules",
...
```

---

`2023-04-25` npm 安装失败，用 yarn 安装报错：

```bash
...
[4/4] 🔨  Building fresh packages...
[12/14] ⠁ electron
[9/14] ⠁ jpegtran-bin
[8/14] ⠁ gifsicle
[10/14] ⠁ optipng-bin
error /Users/chemputer/github/react/node_modules/gifsicle: Command failed.
Exit code: 1
Command: node lib/install.js
Arguments:
Directory: /Users/chemputer/github/react/node_modules/gifsicle
Output:
⚠ connect ECONNREFUSED 0.0.0.0:443
  ⚠ gifsicle pre-build test failed
  ℹ compiling from source
  ✖ Error: Command failed: /bin/sh -c autoreconf -ivf
/bin/sh: autoreconf: command not found


    at /Users/chemputer/github/react/node_modules/bin-build/node_modules/execa/index.js:231:11
```

是因为 React 的依赖包中依赖了 autoconf 这个包，我们使用 Homebrew 来安装。

```bash
brew install autoconf automake libtool
```

安装完成之后执行 `yarn install` 继续报错

```bash
[4/4] 🔨  Building fresh packages...
[12/14] ⠁ electron
[10/14] ⠁ optipng-bin
[9/14] ⠁ jpegtran-bin
[8/14] ⠁ gifsicle
error /Users/chemputer/github/react/node_modules/optipng-bin: Command failed.
Exit code: 1
Command: node lib/install.js
Arguments:
Directory: /Users/chemputer/github/react/node_modules/optipng-bin
Output:
⚠ tunneling socket could not be established, cause=connect EHOSTUNREACH 0.0.30.210:80 - Local (192.168.5.20:50425)
  ⚠ optipng pre-build test failed
  ℹ compiling from source
  ✖ Error: Command failed: /bin/sh -c make install
pngrtran.c:99:1: warning: unused function 'png_rtran_ok' [-Wunused-function]
png_rtran_ok(png_structrp png_ptr, int need_IHDR)
^
1 warning generated.
pngrutil.c:3536:20: warning: performing pointer subtraction with a null pointer has undefined behavior [-Wnull-pointer-subtraction]
                   png_isaligned(dp, png_uint_16) &&
                   ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

报错 [Unknown system error -86 when building on Apple Silicon](https://github.com/imagemin/optipng-bin/issues/117) 是因为 `M1 Apple Silicon` 的问题，需要执行

```bash
CPPFLAGS=-DPNG_ARM_NEON_OPT=0 npm install --force
```

### 3. `npm run build` 开始构建

在 `package.json` 中找到执行构建的文件

```json
...
  "scripts": {
    "build": "node ./scripts/rollup/build.js",
    "build-combined": "node ./scripts/rollup/build-all-release-channels.js",
    ...
  }
```

修改 `./scripts/rollup/build.js` 文件中的 `sourcemap` 选项用以生成 `sourcemap`

```js
function getRollupOutputOptions(
  outputPath,
  format,
  globals,
  globalName,
  bundleType
) {
  const isProduction = isProductionBundleType(bundleType);

  return {
    file: outputPath,
    format,
    globals,
    freeze: !isProduction,
    interop: false,
    name: globalName,
    sourcemap: true, // 修改之前是 sourcemap: false,
    esModule: false,
  };
}
```

执行 `npm run build` （若没有下面的报错则中断命令并跳过这一步）

> warning 报错

```bash
➜  react git:(main) ✗ npm run build

> @ build /Users/wyl/github/react
> node ./scripts/rollup/build.js

internal/modules/cjs/loader.js:883
  throw err;
  ^

Error: Cannot find module 'babel-code-frame'
Require stack:
- /Users/wyl/github/react/scripts/rollup/build.js
    at Function.Module._resolveFilename (internal/modules/cjs/loader.js:880:15)
    at Function.Module._load (internal/modules/cjs/loader.js:725:27)
    at Module.require (internal/modules/cjs/loader.js:952:19)
    at require (internal/modules/cjs/helpers.js:88:18)
    at Object.<anonymous> (/Users/wyl/github/react/scripts/rollup/build.js:23:19)
    at Module._compile (internal/modules/cjs/loader.js:1063:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:1092:10)
    at Module.load (internal/modules/cjs/loader.js:928:32)
    at Function.Module._load (internal/modules/cjs/loader.js:769:14)
    at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:72:12) {
  code: 'MODULE_NOT_FOUND',
  requireStack: [ '/Users/wyl/github/react/scripts/rollup/build.js' ]
}
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! @ build: `node ./scripts/rollup/build.js`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the @ build script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/wyl/.npm/_logs/2022-08-09T09_23_10_249Z-debug.log
```

解决方案：`yarn`

🤔 挺奇怪的，用 `yarn` 重新安装一下就可以正常执行 `npm run build`

再次执行 `npm run build`

> warning 报错

```bash
➜  react git:(main) ✗ npm run build

> @ build /Users/wyl/github/react
> node ./scripts/rollup/build.js

 BUILDING  react.development.js (umd_dev)

Sourcemap is likely to be incorrect: a plugin (at position 6) was used to transform files, but didn't generate a sourcemap for the transformation. Consult the plugin documentation for help

npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! @ build: `node ./scripts/rollup/build.js`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the @ build script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/wyl/.npm/_logs/2022-08-09T09_45_56_927Z-debug.log
```

**原因**：过程中需要很多插件一步一步进行构建，但是有一些插件没有生成 `sourcemap` 文件，导致过程中断，构建失败。

**解决方案**：在 `getPlugins` 函数中注释掉这些插件即可。

```js
1; // Remove 'use strict' from individual source files.
// {
//   transform(source) {
//     return source.replace(/['"]use strict["']/g, '');
//   },
// },

2; // Apply dead code elimination and/or minification.
// isProduction &&
// closure(
//   Object.assign({}, closureOptions, {
//     // Don't let it create global variables in the browser.
//     // https://github.com/facebook/react/issues/10909
//     assume_function_wrapper: !isUMDBundle,
//     renaming: !shouldStayReadable,
//   })
// ),

3; // HACK to work around the fact that Rollup isn't removing unused, pure-module imports.
// Note that this plugin must be called after closure applies DCE.
// isProduction && stripUnusedImports(pureExternalModules),

4; // Add the whitespace back if necessary.
// shouldStayReadable &&
// prettier({
//   parser: 'babel',
//   singleQuote: false,
//   trailingComma: 'none',
//   bracketSpacing: true,
// }),

5; // License and haste headers, top-level `if` blocks.
// {
//   renderChunk(source) {
//     return Wrappers.wrapBundle(
//       source,
//       bundleType,
//       globalName,
//       filename,
//       moduleType,
//       bundle.wrapWithModuleBoundaries
//     );
//   },
// },
```

再次执行 `npm run build` 会在 `build/` 文件夹下得到一系列带有 `sourcemap` 的打包文件。

```bash
➜  react git:(main) ✗ tree build/node_modules/react/umd
build/node_modules/react/umd
├── react.development.js         ⬅️
├── react.development.js.map     ⬅️
├── react.production.min.js
├── react.production.min.js.map
├── react.profiling.min.js
└── react.profiling.min.js.map

0 directories, 6 files
```

```bash
➜  react git:(main) ✗ tree build/node_modules/react-dom/umd
build/node_modules/react-dom/umd
├── react-dom-server-legacy.browser.development.js
├── react-dom-server-legacy.browser.development.js.map
├── react-dom-server-legacy.browser.production.min.js
├── react-dom-server-legacy.browser.production.min.js.map
├── react-dom-server.browser.development.js
├── react-dom-server.browser.development.js.map
├── react-dom-server.browser.production.min.js
├── react-dom-server.browser.production.min.js.map
├── react-dom-test-utils.development.js
├── react-dom-test-utils.development.js.map
├── react-dom-test-utils.production.min.js
├── react-dom-test-utils.production.min.js.map
├── react-dom.development.js        ⬅️
├── react-dom.development.js.map    ⬅️
├── react-dom.production.min.js
├── react-dom.production.min.js.map
├── react-dom.profiling.min.js
└── react-dom.profiling.min.js.map

0 directories, 18 files
```

重点关注下面 `4` 个文件

- `react.development.js`
- `react.development.js.map`
- `react-dom.development.js`
- `react-dom.development.js.map`

## 第二步：创建调试应用

### 1. 用 `CRA` 创建一个 `APP`

```bash
npx create-react-app debug-react-app
```

调试源码没必要用 `ts`，避免一些不必要的 `ts` 类型检查错误

### 2. 暴露配置

`CRA` 创建的应用还是引用的打包之后的文件并且不能自定义配置，所以需要修改。

执行 `npm run reject` 暴露配置项。

### 3. 修改配置

在 `config/webpack.config.js` 文件中新增 [`externals`](https://webpack.js.org/configuration/externals/#externals) 👈 属性。这样可以单独加载 `react`，`react-dom`。

```js
...
    ].filter(Boolean),
    // Turn off performance processing because we utilize
    // our own hints via the FileSizeReporter
    performance: false,
    externals: {
      react: 'React',
      'react-dom': 'ReactDOM'
    }
  };
};
```

### 4. 将刚才生成的下述文件粘贴到 `public` 目录下，并在 `index.html` 中加载

- `react.development.js`
- `react.development.js.map`
- `react-dom.development.js`
- `react-dom.development.js.map`

```bash
➜  debug-react-app git:(master) ✗ tree public
public
├── favicon.ico
├── index.html
├── logo192.png
├── logo512.png
├── manifest.json
├── react-dom.development.js        ⬅️
├── react-dom.development.js.map    ⬅️
├── react.development.js            ⬅️
├── react.development.js.map        ⬅️
└── robots.txt

0 directories, 10 files
```

```html
...
      To create a production bundle, use `npm run build` or `yarn build`.
    -->
  </body>
</html>

<script src="%PUBLIC_URL%/react.development.js"></script>
<script src="%PUBLIC_URL%/react-dom.development.js"></script>
```

### 5. 粘贴 `react` 源码到调试应用目录下

```bash
mv /p/a/t/h/react .
```

### 6. 关联 `react` 源码

打开 `react-dom.development.js.map` 文件发现 `sources` 属性的文件前缀是 `../../../../packages`。修改成当下的 `react` 源码路径就可以了。

```js
{
    "version": 3,
    "file": "react-dom.development.js",
    "sources": [
        "../../../../packages/shared/ReactSharedInternals.js",
        "../../../../packages/shared/consoleWithStackDev.js",
        "../../../../packages/react-reconciler/src/ReactWorkTags.js",
        ...
```

解决思路：修改配置项并重新构建刚才生成的 `4` 个文件

在刚才转移的 `react源码` 文件中找到 `/react/scripts/rollup/build.js` 新增 `sourcemapPathTransform` 方法，将路径修改为自己的项目路径

```js
function getRollupOutputOptions(
  outputPath,
  format,
  globals,
  globalName,
  bundleType
) {
  const isProduction = isProductionBundleType(bundleType);

  return {
    file: outputPath,
    format,
    globals,
    freeze: !isProduction,
    interop: false,
    name: globalName,
    // sourcemap: false,
    sourcemap: true,
    esModule: false,
    sourcemapPathTransform(relativeSourcePath, sourcemapPath) { ⬅️
      return relativeSourcePath.replace('../../../../packages', '/Users/wyl/github/debug-react-app/react/packages')
    }
  };
}
```

再次打开新生成的 `react-dom.development.js.map` 文件可以看到 `sources` 路径已经修改完成。

```js
{
    "version": 3,
    "file": "react.development.js",
    "sources": [
        "/Users/wyl/github/debug-react-app/react/packages/shared/ReactVersion.js",
        "/Users/wyl/github/debug-react-app/react/packages/shared/ReactSymbols.js",
        "/Users/wyl/github/debug-react-app/react/packages/react/src/ReactCurrentDispatcher.js",
    ...
```

### 7. 把之前 `public` 目录下的这 `4` 个文件用新生成的替换掉

- `react.development.js`
- `react.development.js.map`
- `react-dom.development.js`
- `react-dom.development.js.map`

## 第三步：创建 `VSCode Debug` 配置文件

在 `VSCode` 左侧栏中点击 `Run and Debug` 按钮。点击 `绿色小箭头` 的 `Add Configuration` 新增配置文件

> `url` 中的端口记得修改成自己的

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch Chrome",
      "request": "launch",
      "type": "chrome",
      "url": "http://localhost:3000",
      "webRoot": "${workspaceFolder}"
    }
  ]
}
```

使用方法：

1. 在项目中执行 `npm run start` 启动

2. 再点击 `绿色小箭头` 启动 `debug`

3. 标记断点，点击对应的 `CALL STACK`

4. 点击 `VSCode` 左侧栏中的 `Explorer` 就可以查看到对应的 `React` 源码文件了

大功告成！🎉

> 但是这种方法有个缺点：直接在源文件中修改代码是无效的，因为最终生效的还是打包之后的文件

## 参考阅读

- [React 技术揭秘 【魔术师卡颂】](https://react.iamkasong.com/)
- [React 源码解析](https://react.jokcy.me/)
