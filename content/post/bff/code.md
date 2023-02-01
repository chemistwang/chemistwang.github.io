---
# layout: post
title: "BFF（二）BFF的落地实现"
subtitle: ""
date: 2023-01-10
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: ""
---

# BFF 的落地实现

[Github 完整项目地址](https://github.com/chemistwang/demo-bff)

## 1. 前置知识

- [聊聊 Node.js RPC（一）— 协议](https://www.yuque.com/egg/nodejs/dklip5)

  `RPC(Remote Procedure Call)` 是远程过程调用的缩写，是一种通信协议，允许程序在不同的计算机上相互调用远程过程，就像调用本地过程一样

- `sofa-rpc-node`：基于 `nodejs` 的一个 `RPC` 框架，支持多种协议
- `Protocol Buffers`(简称 `protobuf`)：`Google`开发的一种数据序列化格式，可以将结构化数据序列化成二进制格式，并能够垮语言使用

## 2. 技术选型

- MySQL
- Redis
- Zookeeper
- Koa

## 3. 环境安装并启动

- macOS 11.6.4
- 本地 `localhost` 运行

```bash
# 安装
brew install mysql
brew install zookeeper
```

```bash
# 启动 zookeeper
zkServer start
```

## 4. 安装 mysql 并创建数据库和表

```sql
-- t_user 表
CREATE TABLE IF NOT EXISTS `t_user`(
   `id` INT UNSIGNED AUTO_INCREMENT,
   `username` VARCHAR(20) NOT NULL,
   `avatar` VARCHAR(100) NOT NULL,
   `password` VARCHAR(100) NOT NULL,
   `phone` VARCHAR(30) NOT NULL,
   PRIMARY KEY ( `id` )
)

-- t_post 表
CREATE TABLE IF NOT EXISTS `t_post`(
   `id` INT UNSIGNED AUTO_INCREMENT,
   `user_id` INT,
   PRIMARY KEY ( `id` )
)
```

## 5. 创建 user 微服务

### 5.1 安装

```bash
mkdir user && npm init -y
npm install mysql2 sofa-rpc-node --save
```

```json
{
  "name": "user",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "nodemon index.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "mysql2": "^3.0.0",
    "sofa-rpc-node": "^2.8.0"
  }
}
```

### 5.2 启动 RPC 服务并发布 user 服务

```bash
touch user/index.js
```

```js
const {
  server: { RpcServer }, // 可以使用它创建RPC服务器
  registry: { ZookeeperRegistry }, // 可以通过它来创建注册中心
} = require("sofa-rpc-node");
const mysql = require("mysql2/promise");
const logger = console;

// 创建一个注册中心，用于注册微服务
const registry = new ZookeeperRegistry({
  // 记录日志用哪个工具
  logger,
  // zookeeper地址
  address: "127.0.0.1:2181",
});

// 创建RPC服务器的实例
// 客户端连接RPC服务器的可以通过zookeeper，也可以直接直连rpcServer
const server = new RpcServer({
  logger,
  registry,
  port: 10000,
});

(async function () {
  //连接mysql数据库
  const connection = await mysql.createConnection({
    host: "localhost",
    user: "root",
    password: "",
    database: "bff",
  });
  // 添加服务接口
  server.addService(
    // 格式约定为域名反转+领域模型的名称
    { interfaceName: "com.chemputer.user" }, // 格式约定为域名反转+领域模型的名称
    {
      async getUserInfo(userId) {
        const sql = `SELECT id,username,avatar,password,phone from user WHERE id=${userId} limit 1`;
        const [rows] = await connection.execute(sql);
        return rows[0];
      },
    }
  );
  // 启动RPC服务
  await server.start();
  // 把启动好的RPC服务注册到zookeeper上
  await server.publish();
  console.log("用户微服务已经启动");
})();
```

### 5.3 客户端

```bash
touch user/client.js
```

```js
const {
  client: { RpcClient }, // 可以使用它创建RPC服务
  registry: { ZookeeperRegistry }, // 可以通过它来创建注册中心
} = require("sofa-rpc-node");
const logger = console;
// 创建一个注册中心，用于注册微服务
const registry = new ZookeeperRegistry({
  // 记录日志用哪个工具
  logger,
  // zookeeper的地址
  address: "127.0.0.1:2181",
});
(async function () {
  // 创建RPC客户端
  const client = new RpcClient({
    logger,
    registry,
  });
  // 创建一个RPC服务器的消费者
  const consumer = client.createConsumer({
    interfaceName: "com.chemputer.user",
  });
  // 等待服务就绪
  await consumer.ready();
  const result = await consumer.invoke("getUserInfo", [1]);
  console.log(result, "result....");
  process.exit(0);
})();
```

## 6. 创建 post 微服务

### 6.1 安装

```bash
mkdir user && npm init -y
npm install mysql2 sofa-rpc-node --save
```

## 7. bff 中间层

```bash
npm i amqplib fs-extra ioredis koa koa-logger koa-router lru-cache sofa-rpc-node
```
