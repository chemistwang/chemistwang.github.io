---
layout: post
title: "「web安全漏洞」前端源代码泄漏"
subtitle: ""
date: 2023-05-06
author: "汪洋龙"
categories: [React]
image: "img/react-bg.jpg"
description: "防止代码泄漏"
tags:
  - "react"
---

默认 `npm run build` 执行打包会生成 `.js.map` 文件。

为了防止源码被还原，解决方案如下：

## 1. 通过设置 script

```json
...
  "scripts": {
    "dev": "craco start",
    "start": "craco start",
    "build-darwin": "export GENERATE_SOURCEMAP=false && craco build",
    "build-win32": "set GENERATE_SOURCEMAP=false && craco build",
    "test": "craco test",
    "eject": "react-scripts eject",
    "lint": "eslint src --ext .js,.ts --cache --fix",
    "prepare": "husky install"
  },
...
```

因为 `windows` 和 `linux/macOS` 操作系统不同，需要 `set` 或 `export` 适配，有些麻烦。

## 2. 通过 CRA 高级设置

在根目录创建 `.env`，设置 `GENERATE_SOURCEMAP` 属性为 `false`。参考 [CRA 高级设置](https://create-react-app.dev/docs/advanced-configuration/)

```
GENERATE_SOURCEMAP = true
```
