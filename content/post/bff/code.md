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

- Koa
- Redis
- MySQL
- Zookeeper
- RabbitMQ

## 3. 环境安装并启动

- macOS `11.6.4`
- 本地 `localhost` 运行

```bash
# 安装
brew install mysql
brew install zookeeper
brew install rabbitmq
```

```bash
# 启动 zookeeper
zkServer start
# 启动 rabbitmq
brew services start rabbitmq
```

## 4. 安装 mysql 并创建数据库和表

### 4.1 创建数据库

```sql
CREATE DATABASE bff;
```

### 4.2 创建表

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
   `title` VARCHAR(100) NOT NULL,
   PRIMARY KEY ( `id` )
)
```

### 4.3 创建模拟数据

```sql
INSERT INTO t_user (id, username, avatar, password, phone) VALUES (1, 'lucus', 'avatar1.png', '111111', '13100000001');
INSERT INTO t_user (id, username, avatar, password, phone) VALUES (2, 'mars', 'avatar2.png', '222222', '13100000002');

INSERT INTO t_post  (id, user_id, title) VALUES (1, 1, '文章1');
INSERT INTO t_post (id, user_id, title) VALUES (2, 1, '文章2');
INSERT INTO t_post (id, user_id, title) VALUES (3, 1, '文章3');
```

## 5. 创建 user 微服务

### 5.1 安装

```bash
mkdir user && npm init -y
npm install mysql2 sofa-rpc-node --save
```

`user/package.json`

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
# 启动服务
npm run dev
```

`user/index.js`

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
        const sql = `SELECT id,username,avatar,password,phone from t_user WHERE id=${userId} limit 1`;
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

`user/client.js`

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

## 6. 创建 post 微服务（同创建 user 微服务）

### 6.1 安装

```bash
mkdir post && npm init -y
npm install mysql2 sofa-rpc-node --save
```

`post/package.json`

```json
{
  "name": "post",
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
    "mysql2": "^3.1.0",
    "sofa-rpc-node": "^2.8.0"
  }
}
```

### 6.2 启动 RPC 服务并发布 post 服务

```bash
touch post/index.js
# 启动服务
npm run dev
```

`post/index.js`

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
  port: 10001,
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
    { interfaceName: "com.chemputer.post" }, // 格式约定为域名反转+领域模型的名称
    {
      async getPostCount(userId) {
        const sql = `SELECT count(*) as postCount from t_post WHERE user_id=${userId}`;
        const [rows] = await connection.execute(sql);
        return rows[0].postCount;
      },
    }
  );
  // 启动RPC服务
  await server.start();
  // 把启动好的RPC服务注册到zookeeper上
  await server.publish();
  console.log("文章微服务已经启动");
})();
```

### 6.3 客户端

```bash
touch post/client.js
```

`post/client.js`

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
    interfaceName: "com.chemputer.post",
  });
  // 等待服务就绪
  await consumer.ready();
  const result = await consumer.invoke("getPostCount", [1]);
  console.log(result, "post result....");
  process.exit(0);
})();
```

## 7. 创建 BFF

### 7.1 安装

```bash
mkdir bff && npm init -y
npm i koa koa-router koa-logger sofa-rpc-node lru-cache ioredis amqplib fs-extra
```

`bff/package.json`

```json
{
  "name": "bff",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "nodemon index.js",
    "start": "pm2 start index.js --name bff"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "amqplib": "^0.10.3",
    "fs-extra": "^11.1.0",
    "ioredis": "^5.2.4",
    "koa": "^2.14.1",
    "koa-logger": "^3.2.1",
    "koa-router": "^12.0.0",
    "lru-cache": "^7.14.1",
    "sofa-rpc-node": "^2.8.0"
  }
}
```

### 7.2 创建 BFF 层 RPC 中间件

```bash
touch bff/middleware/rpc.js
```

`bff/middleware/rpc.js`

```js
const {
  client: { RpcClient }, // 可以使用它创建RPC服务器
  registry: { ZookeeperRegistry }, // 可以通过它来创建注册中心
} = require("sofa-rpc-node");

const logger = console;

const rpcMiddleware = (options = {}) => {
  return async function (ctx, next) {
    // 创建 zookeeperRegistry 类的实例，用于管理服务发现和注册
    const registry = new ZookeeperRegistry({
      // 记录日志用哪个工具
      logger,
      // zookeeper 的地址
      address: "127.0.0.1:2181",
    });
    // 创建 RpcClient 类的实例，用于发送 rpc 请求
    const client = new RpcClient({
      logger,
      registry,
    });
    const interfaceNames = options.interfaceNames || [];
    const rpcConsumers = {};
    for (let i = 0; i < interfaceNames.length; i++) {
      const interfaceName = interfaceNames[i];
      // 使用 RpcClient 的 createConsumer 方法创建 rpc 消费者
      const consumer = client.createConsumer({ interfaceName });
      // 等待 rpc 消费者准备完毕
      await consumer.ready();
      rpcConsumers[interfaceName.split(".").pop()] = consumer;
    }
    ctx.rpcConsumers = rpcConsumers;
    await next();
  };
};

module.exports = rpcMiddleware;
```

### 7.3 启动 BFF 服务

```bash
touch bff/index.js
# 启动服务
npm run dev
```

`bff/index.js`

```js
const Koa = require("koa");
const router = require("koa-router")();
const logger = require("koa-logger");
const rpcMiddleware = require("./middleware/rpc");
const app = new Koa();

app.use(logger());
app.use(
  rpcMiddleware({
    // 配置 rpc 中间件的参数，表示要调用的 rpc 接口名称
    interfaceNames: ["com.chemputer.user", "com.chemputer.post"],
  })
);

router.get("/", async (ctx) => {
  const userId = ctx.query.userId;
  const {
    rpcConsumers: { user, post },
  } = ctx;
  const [userInfo, postCount] = await Promise.all([
    user.invoke("getUserInfo", [userId]),
    post.invoke("getPostCount", [userId]),
  ]);
  ctx.body = {
    userInfo,
    postCount,
  };
});

app.use(router.routes()).use(router.allowedMethods());
app.listen(3000, () => {
  console.log(" 🚀 bff server is running at 3000");
});
```

浏览器访问

```
http://localhost:3000/?userId=1
```

## 8. 数据处理

对数据进行个性化的 `裁剪` `脱敏` `适配`

`bff/index.js`

```js
...
router.get("/", async (ctx) => {
  const userId = ctx.query.userId;
  const {
    rpcConsumers: { user, post },
  } = ctx;
  const [userInfo, postCount] = await Promise.all([
    user.invoke("getUserInfo", [userId]),
    post.invoke("getPostCount", [userId]),
  ]);
  // 数据的裁剪，把不需要的信息和字符裁剪掉，不返回给客户端
  delete userInfo.password;
  // 数据脱敏
  userInfo.phone = userInfo.phone.replace(/(\d{3})\d{4}(\d{4})/, "$1****$2");
  // 数据适配
  userInfo.avatar = "http://www.chemputer.top/" + userInfo.avatar;
  ctx.body = {
    userInfo,
    postCount,
  };
});
...
```

## 9. 缓存

- BFF 作为前端应用和后端系统之间的抽象层，承担了大量的请求转发和数据转换工作。使用多级缓存可以帮助 BFF 减少对后端系统的访问，从而提高应用的响应速度
- 当 BFF 收到一个请求时，首先会检查内存缓存中是否存在对应的数据，如果有就直接返回数据。如果内存缓存中没有数据，就会检查 Redis 缓存，如果 Redis 缓存中有数据就返回数据，并将数据写入内存缓存。如果本地缓存中也没有数据，就会向后端系统发起请求，并将数据写入 Redis 缓存和内存缓存

### 9.1 多级缓存

- 多级缓存（`multi-level cache`）是指系统中使用了多个缓存层来存储数据的技术。这些缓存层的优先级通常是依次递减的，即最快的缓存层位于最顶层，最慢的缓存层位于最底层

### 9.2 LRU

- LRU（`Least Recently Used`）是一种常用的高速缓存淘汰算法，它的原理是将最近使用过的数据或页面保留在缓存中，而最少使用的数据或页面将被淘汰。这样做的目的是为了最大化缓存的命中率，即使用缓存尽可能多地满足用户的请求

### 9.3 Redis

- `Redis` 是一种开源的内存数据存储系统，可以作为数据库、缓存和消息中间件使用
- `Redis` 运行在内存中，因为它的读写速度非常快
- `ioredis` 是一个基于 `Node.js` 的 `Redis` 客户端，提供了对 `Redis` 命令的高度封装和支持

### 9.4 BFF 使用缓存

```bash
touch bff/middleware/cache.js
```

`bff/middleware/cache.js`

```js
const Redis = require("ioredis");
const LRUCache = require("lru-cache");
class CacheStore {
  constructor() {
    this.stores = [];
  }
  add(store) {
    this.stores.push(store);
    return this;
  }
  async set(key, value) {
    for (const store of this.stores) {
      await store.set(key, value);
    }
  }
  async get(key) {
    for (const store of this.stores) {
      const value = await store.get(key);
      if (value !== undefined) return value;
    }
  }
}
class MemoryStore {
  constructor() {
    this.cache = new LRUCache({
      max: 100,
      //一般来说越上层的过期时间越短
      ttl: 1000, //设置过期时间
    });
  }
  async set(key, value) {
    this.cache.set(key, value);
  }
  async get(key) {
    return this.cache.get(key);
  }
}

class RedisStore {
  constructor(options = { host: "localhost", port: "6379" }) {
    this.client = new Redis(options);
  }
  async set(key, value) {
    await this.client.set(key, JSON.stringify(value));
    //await this.client.pexpire(key, 60);
  }
  async get(key) {
    const value = await this.client.get(key);
    return value ? JSON.parse(value) : undefined;
  }
}
//创建一个缓存实例
const cacheStore = new CacheStore();
//添一些缓存层
cacheStore.add(new MemoryStore());
cacheStore.add(new RedisStore());

const cacheMiddleware = () => {
  return async function (ctx, next) {
    ctx.cache = cacheStore;
    await next();
  };
};
module.exports = cacheMiddleware;
```

## 10. 消息队列处理日志

- 消息队列（`Message Queue`）用于在分布式系统中传递数据。它的特点是可以将消息发送者和接收者解耦，使得消息生产者和消息消费者可以独立的开发和部署

### 10.1 引入原因

- 在 BFF 中使用消息队列（`message queue`）有几个原因
  - `大并发`：消息队列可以帮助应对大并发的请求，BFF 可以将请求写入消息队列，然后后端服务可以从消息队列中读取请求并处理
  - `解耦`：消息队列可以帮助解耦 BFF 和后端服务，BFF 不需要关心后端服务的具体实现，只需要将请求写入消息队列，后端服务负责从消息队列中读取请求并处理
  - `异步`：消息队列可以帮助实现异步调用，BFF 可以将请求写入消息队列，然后立即返回响应给前端应用，后端服务在后台处理请求
  - `流量削峰`：消息队列可以帮助流量削峰，BFF 可以将请求写入消息队列，然后后端服务可以在合适的时候处理请求，从而缓解瞬时高峰流量带来的压力

### 10.2 RabbitMQ

- `RabbitMQ` 是一个消息代理，它可以用来在消息生产者和消息消费者之间传递消息
- `RabbitMQ` 的工作流程如下：
  - 消息生产者将消息发送到 `RabbitMQ` 服务器
  - `RabbitMQ` 服务器将消息保存到队列中
  - 消息消费者从队列中读取消息
  - 当消息消费者处理完消息后 `RabbitMQ` 服务器将消息删除

### 10.3 BFF 使用 MQ

1. 创建 mq 中间件

```bash
touch bff/middleware/mq.js
```

`bff/middleware/mq.js`

```js
const amqplib = require("amqplib");
const mqMiddleware = (options = {}) => {
  return async function (ctx, next) {
    //连接 MQ服务器
    const mqClient = await amqplib.connect(options.url || "amqp://localhost");
    //创建一个通道
    const logger = await mqClient.createChannel();
    //创建一个名称为logger的队列，如果已经存在，不会重复创建
    await logger.assertQueue("logger");
    ctx.channels = {
      logger,
    };
    await next();
  };
};
module.exports = mqMiddleware;
```

2. 接入到 bff 层

`bff/index.js`

```js
router.get('/', async (ctx) => {
   ...
  ctx.channels.logger.sendToQueue(
    "logger",
    Buffer.from(
      JSON.stringify({
        method: ctx.method,
        path: ctx.path,
        userId,
      })
    )
  );
   ...
})
```

3. 启动写入日志服务

```bash
mkdir write-logger && npm init -y
npm i amqplib fs-extra
```

`write-logger/package.json`

```json
{
  "name": "write-logger",
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
    "amqplib": "^0.10.3",
    "fs-extra": "^11.1.0"
  }
}
```

`write-logger/index.js`

```js
const amqplib = require("amqplib");
const fs = require("fs-extra");
(async function () {
  //连接 MQ服务器
  const mqClient = await amqplib.connect("amqp://localhost");
  //创建一个通道
  const logger = await mqClient.createChannel();
  //创建一个名称为logger的队列，如果已经存在，不会重复创建
  await logger.assertQueue("logger");
  logger.consume("logger", async (event) => {
    await fs.appendFile("./logger.txt", event.content.toString() + "\n");
  });
})();
```
