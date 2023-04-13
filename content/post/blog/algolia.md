---
# layout: post
title: "Hugo 配置 algolia"
subtitle: ""
date: 2023-04-13
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "赋予博客搜索功能"
tags:
  - "blog"
---

# Hugo 配置 algolia

> 参考：[Static site search with Hugo + Algolia](https://forestry.io/blog/search-with-algolia-in-hugo/)

## 1. 注册并配置 algolia

### 1.1 注册并创建应用

![step1](/post/blog/step1.jpg)

### 1.2 选择一个数据中心

![step2](/post/blog/step2.jpg)

### 1.3 同意条款

![step3](/post/blog/step3.jpg)

### 1.4 创建索引

类似于一个命名空间

![step4](/post/blog/step4.jpg)

### 1.5 查看对应的 API Keys 并保存

![apikey](/post/blog/apikey.jpg)

## 2. 生成并上传索引文件

### 2.1 生成索引文件

执行 `hugo` 命令生成索引

```bash
hugo
```

当前我的主题会在 `public` 文件下生成 `algolia.json` 文件

### 2.2 创建 .env 文件

```bash
# 需要将双括号一并替换
ALGOLIA_APP_ID={{ YOUR_APP_ID }}
ALGOLIA_ADMIN_KEY={{ YOUR_ADMIN_KEY }}
# 和在 algolia 创建的索引保持一致
ALGOLIA_INDEX_NAME={{ YOUR_INDEX_NAME }}
# 我的是 public/algolia.json
ALGOLIA_INDEX_FILE={{ PATH/TO/algolia.json }}
```

> 记得在 .gitignore 中添加 .env，防止信息泄露

### 2.3 在当前博客项目安装 `atomic-algolia` 自动上传索引文件

```bash
$ npm init
$ npm i atomic-algolia
```

### 2.4 在 package.json 文件中新增 script

```json
...
  "scripts": {
    "algolia": "atomic-algolia"
  },
...
```

### 2.5 上传

直接执行命令将生成的索引文件上传至 `algolia` 服务

```bash
npm run algolia
```

## 3. 配置 Hugo config.toml 开启搜索

```bash
# algolia site search
algolia_search = true
algolia_appId = {{ YOUR_APP_ID }}
algolia_indexName = {{ YOUR_INDEX_NAME }}
algolia_apiKey = {{ YOUR_SEARCH_ONLY_KEY }}
```
