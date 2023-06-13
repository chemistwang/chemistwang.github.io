---
# layout: post
title: "Nginx配置http2"
subtitle: ""
date: 2022-07-18 10:49:10
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "是时候升级了"
---

## 配置 http2

[http2 速度渲染比较](https://http2.akamai.com/demo) 👈

鉴于 `http2` 协议的诸多优点，想升级一下自己的静态网页。

官方 `Nginx` 默认不包含 `h2` 模块，需要加入参数编译

### 1. 查看编译模块, 发现只有 `ssl` 模块

```bash
[root@smartcentos ~]# nginx -V
nginx version: nginx/1.20.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --with-http_ssl_module
```

> 最后一行的 `--with-http_ssl_module`

### 2. 查看 openssl 版本

```bash
openssl version # 查看openssl版本
OpenSSL 1.0.2k-fips  26 Jan 201
```

### 3. 下载比较新的 `openssl` 版本（尝试过使用最新 `3.x` 版本，但是失败）

```bash
wget https://www.openssl.org/source/openssl-1.1.1q.tar.gz
# 解压
tar -zxvf openssl-1.1.1q.tar.gz -C /opt/module/
```

### 4. 编译 `--with-http_v2_module` 模块

```bash
cd /opt/module/nginx-1.20.1 # 找到安装地址
./configure --help # 查看是否支持 --with-http_v2_module
 ./configure --with-http_v2_module --with-http_ssl_module --with-openssl=/opt/module/openssl-1.1.1q # 别忘记之前添加的模块
gmake && gmake install
```

### 5. 检查

```bash
[root@smartcentos nginx-1.20.1]# nginx -V
nginx version: nginx/1.20.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
built with OpenSSL 1.1.1q  5 Jul 2022
TLS SNI support enabled
configure arguments: --with-http_v2_module --with-http_ssl_module --with-openssl=/opt/module/openssl-1.1.1q
```

> 最后一行 `--with-http_v2_module --with-http_ssl_module --with-openssl=/opt/module/openssl-1.1.1q`

### 6. 修改 `nginx.conf`

```
server {
    listen       80;
    # listen       443 ssl;
    listen       443 ssl http2; 👈
    server_name notebook.chemputer.top;

    ssl_certificate      /etc/letsencrypt/live/notebook.chemputer.top/fullchain.pem;
    ssl_certificate_key  /etc/letsencrypt/live/notebook.chemputer.top/privkey.pem;

    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;

    location / {
        root   /root/notebook/dist/;
        index  index.html index.htm;
    }
}
```

### 7. 重启 `nginx`

```bash
nginx -s reload
```

重启之后可以通过 [`https://myssl.com/http2_check.html`](https://myssl.com/http2_check.html) 👈 检测是否支持 `HTTP2`
