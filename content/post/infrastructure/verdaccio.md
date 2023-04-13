---
# layout: post
title: "前端基建之（七）npm私服：Verdaccio"
subtitle: ""
date: 2023-04-10
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "npm也可以私有化部署"
tags:
  - "infrastructure"
---

# 前端基建之（七）npm 私服：Verdaccio

## 1. 私服方案

| 私服方案     | 支持仓库     | github                                                   | 开发语言   |
| ------------ | ------------ | -------------------------------------------------------- | ---------- |
| Nexus 开源版 | maven,npm... | [nexus-public](https://github.com/sonatype/nexus-public) | Java       |
| Sinopia      | npm          | [sinopia](https://github.com/rlidwka/sinopia)            | JavaScript |
| Verdaccio    | npm          | [verdaccio](https://github.com/verdaccio/verdaccio)      | JavaScript |
| cnpm.org     | npm          | [cnpmjs](https://github.com/cnpm/cnpmjs.org)             | JavaScript |
| cnpmcore     | npm          | [cnpmcore](https://github.com/cnpm/cnpmcore)             | JavaScript |

[![Star History Chart](https://api.star-history.com/svg?repos=sonatype/nexus-public,cnpm/cnpmjs.org,cnpm/cnpmcore,verdaccio/verdaccio,rlidwka/sinopia&type=Date)](https://star-history.com/#sonatype/nexus-public&cnpm/cnpmjs.org&cnpm/cnpmcore&verdaccio/verdaccio&rlidwka/sinopia&Date)

说明：

`cnpm.org` 停止更新后，团队使用 `TypeScript` 重构并新建了 `cnpmcore` 仓库

`Sinopia` 停止更新后，社区大佬 `fork` 了一份并更名为 `Verdaccio`

## 2. Verdaccio

哪个 star 多就用哪个

### 2.1 安装运行

在服务器上直接用 `Docker` 部署，并开启对应安全组

```bash
docker run -it --rm --name verdaccio -d -p 4873:4873 verdaccio/verdaccio
```

因为是 `alpine` 构建的，所以需要用 `/bin/sh` 进入容器

```bash
docker exec -it verdaccio /bin/sh
```

可以成功访问，但有 `两` 个问题：

- 开放式的环境并不安全，既然是私服需要有一定的权限控制
- 需要挂载卷保存数据，防止容器停止数据丢失

### 2.2 优化

#### 1. 权限修改

默认配置文件：[Github verdaccio 5.x Docker](https://github.com/verdaccio/verdaccio/blob/5.x/conf/docker.yaml)

```yaml
storage: /verdaccio/storage/data
plugins: /verdaccio/plugins
web:
  # 修改标题
  title: Chemputer NPM
auth:
  htpasswd:
    file: /verdaccio/storage/htpasswd
    # 禁止注册
    max_users: -1
uplinks:
  npmjs:
    url: https://registry.npmjs.org/
packages:
  "@*/*":
    # 必须登录才能访问
    access: $authenticated
    publish: $authenticated
    proxy: npmjs
  "**":
    # 必须登录才能访问
    access: $authenticated
    publish: $authenticated
    unpublish: $authenticated
    proxy: npmjs
log: { type: stdout, format: pretty, level: http }
```

#### 2. 创建文件夹存放 conf 文件

服务器创建 `/root/verdaccio` 文件夹存放 `conf` `plugins ` `storage`

```bash
drwxr-xr-x 2 root  root  4096 Apr 11 11:25 conf
drwxr-xr-x 2 root  root  4096 Apr 11 10:43 plugins
drwxr-xr-x 3 10001 65533 4096 Apr 11 11:08 storage
.
|-- conf
|   -- config.yaml
|-- plugins
|-- storage
```

需要设置权限 `chown -R 10001:65533 /root/verdaccio/storage/`，否则 docker logs 会报错

```bash
...
 error--- unexpected error: EACCES: permission denied, mkdir '/verdaccio/storage/data/my-npm-package'
...
```

#### 3. 挂载卷

```bash
docker run -it --rm --name verdaccio -d -p 4873:4873 \
-v /root/verdaccio/conf:/verdaccio/conf \
-v /root/verdaccio/storage:/verdaccio/storage/data \
-v /root/verdaccio/plugins:/verdaccio/plugins \
verdaccio/verdaccio
```

#### 4. 重启

修改完配置文件之后重启即可生效

```bash
docker restart verdaccio
```
