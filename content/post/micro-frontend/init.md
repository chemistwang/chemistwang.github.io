---
layout: post
title: "微前端【qiankun】小试（一）主应用和子应用初始化"
subtitle: ""
date: 2023-01-04
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "分久必合，合久必分"
tags:
  - qiankun
---

# 主项目子项目初始化

## 前期准备

1. 分别创建主应用和子应用

```bash
# 创建 base-react
npx create-react-app base-react
# 创建 mini-react18
npx create-react-app mini-react18
# 创建 mini-vue2
vue create mini-vue2
```

2. 创建 `.env` 指定具体端口，方便管理

| 应用         | 端口  |
| ------------ | ----- |
| base-react   | 11110 |
| mini-react18 | 11111 |
| mini-vue2    | 11112 |

> 说明：chrome 对于端口有部分的限制，指定非法端口号会出现 `ERR_UNSAFE_PORT`，可搜索关键词 `Google Chrome 默认非安全端口列表`

## base-react

### 描述

- 主应用
- 不限技术栈，只需要提供一个容器 DOM
- 注册微应用并 start 即可

### 步骤

1. 安装 `qiankun`

```bash
yarn add qiankun # 或者 npm i qiankun -S
```

2. 在 `App.tsx` 中创建容器

```tsx
function App() {
  return (
    <div className="App">
      <h1>Base React App</h1>
      <div id="mini-react18"></div>
      <div id="mini-vue2"></div>
    </div>
  );
}

export default App;
```

3. 在 `index.tsx` 中注册并启动

```js
import { registerMicroApps, start } from "qiankun";

registerMicroApps([
  {
    name: "react app 18",
    entry: "//localhost:11111",
    container: "#mini-react18",
    activeRule: "/mini-react18",
  },
  {
    name: "vue app 2",
    entry: "//localhost:11112",
    container: "#mini-vue2",
    activeRule: "/mini-vue2",
  },
]);
// 启动 qiankun
start();
```

## mini-react18

### 描述

- 微应用 1（react18）

### 步骤

1. 新建 `src/public-path.js`

```js
if (window.__POWERED_BY_QIANKUN__) {
  __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
}
```

2. 新建 `src/types/index.d.ts`

因为项目基于 ts 编译，需要手动指定 `POWERED_BY_QIANKUN`，否则会报错

```ts
declare global {
  interface Window {
    __POWERED_BY_QIANKUN__: string;
  }
}

export {};
```

3. 利用 `react-app-rewired` 重写 `webpack` 配置

安装

```bash
npm i react-app-rewired
```

根目录新建 `config-overrides.js` 文件 （文件名是规定好的）

```js
const { name } = require("./package");

module.exports = {
  webpack: (config) => {
    config.output.library = `${name}-[name]`;
    config.output.libraryTarget = "umd";
    // config.output.jsonpFunction = `webpackJsonp_${name}`;
    config.output.globalObject = "window";

    return config;
  },

  devServer: (_) => {
    const config = _;

    config.headers = {
      "Access-Control-Allow-Origin": "*",
    };
    config.historyApiFallback = true;
    config.hot = false;
    config.watchContentBase = false;
    config.liveReload = false;

    return config;
  },
};
```

修改 `package.json`

```json
  "scripts": {
    "start": "react-app-rewired start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
```

4. 在 `index.tsx` 中导出子应用生命周期

```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import "./index.css";
import App from "./App";
import reportWebVitals from "./reportWebVitals";

// const root = ReactDOM.createRoot(
//   document.getElementById('root') as HTMLElement
// );
// root.render(
//   <React.StrictMode>
//     <App />
//   </React.StrictMode>
// );

// let root: ReactDOM.Root;

function render(props: any) {
  const { container } = props;
  const root = container
    ? ReactDOM.createRoot(container.querySelector("#root") as HTMLElement)
    : ReactDOM.createRoot(document.getElementById("root") as HTMLElement);

  root.render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  );
}

if (!window.__POWERED_BY_QIANKUN__) {
  render({});
}

export async function bootstrap() {
  console.log("[react18] react app bootstraped");
}

export async function mount(props: any) {
  console.log("[react18] props from main framework", props);
  render(props);
}

export async function unmount(props: any) {
  const { container } = props;
  const root = container
    ? ReactDOM.createRoot(container.querySelector("#root") as HTMLElement)
    : ReactDOM.createRoot(document.getElementById("root") as HTMLElement);
  root.unmount();
}

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

5. 解决静态资源不加载

在 `index.tsx` 顶部引入 `public-path`

```tsx
import "./public-path";
...
```

## mini-vue2

### 描述

- 微应用 2（vue2.0）

### 步骤

1. 新建 `src/public-path.js`

```js
if (window.__POWERED_BY_QIANKUN__) {
  __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
}
```

2. 修改 `main.js`

```js
import "./public-path";
import Vue from "vue";
import App from "./App.vue";

Vue.config.productionTip = false;

// new Vue({
//   render: h => h(App),
// }).$mount('#app')

// let router = null;
let instance = null;
function render(props = {}) {
  const { container } = props;
  // router = new VueRouter({
  //   base: window.__POWERED_BY_QIANKUN__ ? "/app-vue/" : "/",
  //   mode: "history",
  //   routes,
  // });

  instance = new Vue({
    // router,
    // store,
    render: (h) => h(App),
  }).$mount(container ? container.querySelector("#app") : "#app");
}

// 独立运行时
if (!window.__POWERED_BY_QIANKUN__) {
  render();
}

export async function bootstrap() {
  console.log("[vue] vue app bootstraped");
}
export async function mount(props) {
  console.log("[vue] props from main framework", props);
  render(props);
}
export async function unmount() {
  instance.$destroy();
  instance.$el.innerHTML = "";
  instance = null;
  // router = null;
}
```

3. 根目录新增 `vue.config.js`

```js
const { name } = require("./package");
module.exports = {
  devServer: {
    headers: {
      "Access-Control-Allow-Origin": "*",
    },
  },
  configureWebpack: {
    output: {
      library: `${name}-[name]`,
      libraryTarget: "umd", // 把微应用打包成 umd 库格式
      jsonpFunction: `webpackJsonp_${name}`,
    },
  },
};
```

## 启动

分别启动 `base-react`、`mini-react18`、`mini-vue2`

```bash
# 启动 base-react
npm run start
# 启动 mini-react18
npm run start
# 启动 mini-vue2
npm run serve
```

通过路由 `http://localhost:11110/mini-react18` 访问 `mini-react18` 子应用，通过路由 `http://localhost:11110/mini-vue2` 访问 `mini-vue2` 子应用
