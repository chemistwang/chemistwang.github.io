---
# layout: post
title: "FRP 内网穿透 SSH 连接树莓派"
subtitle: "内网穿透 树莓派"
date: 2021-10-03 19:00:00
author: "汪洋龙"
tags:
  - raspberrypi
  - 内网穿透
categories: [Tech]
image: "img/frp-raspberrypi-bg.jpg"
excerpt: ""
description: ""
published: true
---

# FRP 内网穿透 SSH 连接树莓派

> 前提：必须有一个有公网 IP 的服务器

## 安装

1. 查看树莓派和云服务器对应架构

```bash
# Raspberry Pi
~ $ uname -a
Linux raspberrypi 5.10.17-v7l+ #1414 SMP Fri Apr 30 13:20:47 BST 2021 armv7l GNU/Linux
# Server
~ $ uname -a
Linux VM-4-5-centos 3.10.0-1160.11.1.el7.x86_64 #1 SMP Fri Dec 18 16:34:56 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```

2. 下载解压对应架构安装包

```bash
# Raspberry Pi
wget https://github.com/fatedier/frp/releases/download/v0.37.1/frp_0.37.1_linux_arm.tar.gz
tar -zxvf frp_0.37.1_linux_arm.tar.gz
# Server
wget https://github.com/fatedier/frp/releases/download/v0.37.1/frp_0.37.1_linux_amd64.tar.gz
tar -zxvf frp_0.37.1_linux_amd64.tar.gz
```

3. 移动路径

```bash
# Raspberry Pi
sudo mv frp_0.37.1_linux_arm /usr/local/frp
# Server
sudo mv frp_0.37.1_linux_amd64 /usr/local/frp
```

## 修改配置文件

树莓派配置文件为 `frpc.ini`，服务器配置文件为 `frps.ini`

```bash
# frps.ini
[common]
bind_port = 7000
# frpc.ini
[common]
server_addr = x.x.x.x # 公网ip
server_port = 7000
[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

## 启动服务

1. 分别启动

```bash
cd /usr/local/frp

# Raspberry Pi
./frpc -c ./frpc.ini
# Server
./frps -c frps.ini
```

2. 测试

```bash
ssh -oPort=6000 pi@x.x.x.x
```

正常情况会连接成功，但是云服务器需要配置相应端口，所以会失败

## 开启安全组

```bash
0.0.0.0/0 TCP:6000
0.0.0.0/0 TCP:7000
```

## 后台启动

```bash
# Raspberry Pi
nohup ./frpc -c ./frpc.ini > frpc.log 2>&1 &
# Server
nohup ./frps -c ./frps.ini > frps.log 2>&1 &
```

## 配置 token

出于安全考虑，可以在服务器和树莓派的配置文件中增加 token 字段

> 参考资料
>
> - [frp#token-authentication](https://github.com/fatedier/frp#token-authentication)

```md
Token Authentication
When specifying authentication_method = token under [common] in frpc.ini and frps.ini - token based authentication will be used.

Make sure to specify the same token in the [common] section in frps.ini and frpc.ini for frpc to pass frps validation
```

需要在 server 和 client 的 `[common]` 下面配置 `authentication_method` 和 `token`

```bash
# frps.ini
[common]
bind_port = 7000
authentication_method = token
token = xxx

# frpc.ini
[common]
server_addr = 127.0.0.1
server_port = 7000
authentication_method = token
token = xxx

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

## 杀掉 nohup 后台进程

```bash
# 查看对应程序PID号
jobs -l
# 杀掉进程
kill -9 PID
```

## 开机自启动

如果启用了 `nohup` 则先杀掉

1. 拷贝启动文件

```bash
# Raspberry Pi
cp /usr/local/frp/systemd/frpc.service /etc/systemd/system/
# Service
cp /usr/local/frp/systemd/frps.service /etc/systemd/system/
```

2. 修改相关路径

```bash
# Raspberry Pi
[Unit]
Description=Frp Client Service
After=network.target

[Service]
Type=simple
User=nobody
Restart=on-failure
RestartSec=5s
# ExecStart=/usr/bin/frpc -c /etc/frp/frpc.ini
# ExecReload=/usr/bin/frpc reload -c /etc/frp/frpc.ini
ExecStart=/usr/local/frp/frpc -c /usr/local/frp/frpc.ini
ExecReload=/usr/local/frp/frpc reload -c /usr/local/frp/frpc.ini

[Install]
WantedBy=multi-user.target

# Server
[Unit]
Description=Frp Server Service
After=network.target

[Service]
Type=simple
User=nobody
Restart=on-failure
RestartSec=5s
# ExecStart=/usr/bin/frps -c /etc/frp/frps.ini
ExecStart=/usr/local/frp/frps -c /usr/local/frp/frps.ini

[Install]
WantedBy=multi-user.target
```

3. 启动

```bash
# 刷新服务
sudo systemctl daemon-reload
# 允许开机启动
sudo systemctl enable frpc.service
# 运行服务
sudo systemctl start frpc.service
# 查看状态
sudo systemctl status frpc.service
```
