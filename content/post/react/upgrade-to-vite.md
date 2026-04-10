---
layout: post
title: "CRA项目升级Vite"
subtitle: "期待Rust工具链"
date: 2022-08-23
author: "汪洋龙"
categories: [React]
image: "img/react-bg.jpg"
tags:
  - "react"
---

截止当前，`46.6k` 的 🌟 直逼 `webpack`。

- [vite 官网](https://vitejs.dev/) 👈
- [vite github](https://github.com/vitejs/vite) 👈

由于维护的老项目体积较大，启动比较慢，同时需要借助 `jenkins` 实现自动化部署，存在多局点多分支，在构建的时候也很慢。所以尝试一下。

我用的是新版本的 `Vite3`，同时使用 `Vite3` 需要 `Node.js` 版本 `14.18+`，`16+`。官方说明： [搭建第一个 Vite 项目](https://cn.vitejs.dev/guide/#scaffolding-your-first-vite-project) 👈

## 设置 node 版本

```bash
nvm alias default v14.18
```

## 安装依赖

```bash
npm i vite -D
npm i @vitejs/plugin-react -D
```

```json
...
  "devDependencies": {
    "@vitejs/plugin-react": "^2.0.1",
    "vite": "^3.0.9"
  }
...
```

> 因为是 `React` 项目，需要 `@vitejs/plugin-react` 插件支撑

## 修改命令

```json
...
  "scripts": {
    "start": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
...
```

## 根目录下创建 vite.config.js

```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
const { resolve } = require("path"); //必须要引入resolve

export default ({ mode }) => {
  return defineConfig({
    plugins: [react()],
    define: {
      "process.env.NODE_ENV": `"${mode}"`,
    },
    resolve: {
      alias: {
        "@api": resolve(__dirname, "/src/api"),
        "@assets": resolve(__dirname, "/src/assets"),
        "@hooks": resolve(__dirname, "/src/hooks"),
        "@common": resolve(__dirname, "/src/common"),
        "@store": resolve(__dirname, "/src/store"),
        "@utils": resolve(__dirname, "/src/utils"),
      },
    },
    server: {
      port: 6666,
      strictPort: true,
      open: true,
      proxy: {
        "/api": {
          target: "http://192.168.0.224:7777",
          changeOrigin: true,
          rewrite: (path) => path.replace(/\/api/, ""),
        },
      },
    },
  });
};
```

[vite 配置项](https://cn.vitejs.dev/config/) 👈

## 修改 public 相关文件

- 将 `/public/index.html` 移动到根目录下
- 将 `%PUBLIC_URL%` 删掉

> 注：为什么这么做，官方有一个描述： [index.html 与项目根目录](https://cn.vitejs.dev/guide/#index-html-and-project-root) 👈

## 启动

```bash
npm run start
```
