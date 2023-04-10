---
layout: post
title: "npm常用命令和nrm管理工具"
subtitle: ""
date: 2023-04-09 15:53:34
author: "汪洋龙"
categories: []
image: "img/coffee-bg.jpg"
description: ""
---

# npm 常用命令和 nrm 管理工具

## 1. 常用命令

### 查看 npm 全局包

```bash
~ npm list -g
/usr/local/lib
├── corepack@0.10.0
├── create-react-app@5.0.1
├── nodemon@2.0.20
├── npm@8.5.0
├── nrm@1.2.5
├── open@8.4.2
└── pm2@5.2.2
```

### 查看 npm 源

```bash
npm config get registry
# 或
npm get registry
```

### 设置 npm 源

```bash
# 淘宝镜像源
npm config set registry http://registry.npm.taobao.org
# npm 官方源
npm config set registry https://registry.npmjs.org
```

这样切换很不方便，用 `nrm` 做管理

## 2. 用 nrm 做源管理

### 2.1 安装

```bash
sudo npm install nrm -g
```

```bash
(base) ➜  ~ nrm ls
/usr/local/lib/node_modules/nrm/cli.js:9
const open = require('open');
             ^

Error [ERR_REQUIRE_ESM]: require() of ES Module /usr/local/lib/node_modules/nrm/node_modules/open/index.js from /usr/local/lib/node_modules/nrm/cli.js not supported.
Instead change the require of index.js in /usr/local/lib/node_modules/nrm/cli.js to a dynamic import() which is available in all CommonJS modules.
    at Object.<anonymous> (/usr/local/lib/node_modules/nrm/cli.js:9:14) {
  code: 'ERR_REQUIRE_ESM'
}
```

发现报错，原因是引用的 `open` 包在 [v9.x](https://www.npmjs.com/package/open/v/9.0.0) 之后改成 `Native ESM`，不再支持 `require` 引入方式。

`Warning: This package is native ESM and no longer provides a CommonJS export. If your project uses CommonJS, you will have to convert to ESM or use the dynamic import() function. Please don't open issues for questions regarding CommonJS / ESM.`

解决方案：指定低版本

```bash
sudo npm install nrm open@8.4.2 -g
```

### 2.2 使用

#### 1. 查看源列表

```bash
(base) ➜  ~ nrm ls

  npm ---------- https://registry.npmjs.org/
  yarn --------- https://registry.yarnpkg.com/
  tencent ------ https://mirrors.cloud.tencent.com/npm/
  cnpm --------- https://r.cnpmjs.org/
  taobao ------- https://registry.npmmirror.com/
  npmMirror ---- https://skimdb.npmjs.com/registry/
```

#### 2. 切换源

```bash
(base) ➜  ~ nrm use taobao


   Registry has been set to: https://registry.npmmirror.com/
```

#### 3. 添加一个源

```bash
(base) ➜  ~ nrm add person http://xxx:4873/

    add registry person success
```

#### 4. 删除

```bash
(base) ➜  ~ nrm del person

    delete registry person success
```

#### 5. 测速

```bash
(base) ➜  ~ nrm test

* npm ------ 778ms
  yarn ----- 842ms
  tencent -- 1455ms
  cnpm ----- 1101ms
  taobao --- 179ms
  npmMirror - 5451ms
```
