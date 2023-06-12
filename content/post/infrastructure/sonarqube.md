---
# layout: post
title: "前端基建之（六）代码 Review：Sonarqube"
subtitle: ""
date: 2022-12-18
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "代码检查工具"
tags:
  - "infrastructure"
---

## 安装部署

`参考资料：` [官方文档](https://docs.sonarqube.org/latest/setup/install-server/)

### macOS

```bash
docker run --name sonarqube --privileged=true -p 11111:9000 -p 11112:9092 sonarqube
```

### ubuntu

#### 1. `root` 创建 `sonarqube` 用户

```bash
# 创建sonarqube并指定目录
useradd -d /home/sonarqube -m sonarqube

# 设置密码
passwd sonarqube
```

#### 2. 给 `sonarqube` 添加 `sudo` 权限

否则会报错 `sonarqube is not in the sudoers file.This incident will be reported`

```bash
# /etc/sudoers文件默认是只读的，对root来说也是，因此需先添加sudoers文件的写权限
chmod u+w /etc/sudoers

# 编辑
vi /etc/sudoers

# User privilege specification
root    ALL=(ALL:ALL) ALL
# 添加
sonarqube       ALL=(ALL:ALL) ALL

# 撤销sudoers文件写权限
chmod u-w /etc/sudoers
```

#### 3. 修改安装文件权限

```bash
# 修改用户
sudo chown -R sonarqube sonarqube-9.1.0.47736
# 修改用户组
sudo chgrp -R sonarqube sonarqube-9.1.0.47736
```

#### 4. 修改配置文件

```bash
# sonar.properties
vi conf/sonar.properties

sonar.jdbc.username=postgres
sonar.jdbc.password=chemputer123

sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube

sonar.web.host=127.0.0.1
sonar.web.port=10004

# 挺重要的，各种问题日志一目了然
sonar.path.logs=logs

sonar.path.data=data
sonar.path.temp=temp
```

```bash
# wrapper.conf
vi conf/wrapper.conf

wrapper.java.command=/usr/bin/java/jdk1.8.0_231/bin/java
```

#### 5. 启动

```bash
bin/linux-x86-64/sonar.sh start
```

发现报错

```
--> Wrapper Started as Daemon
Launching a JVM...
JVM exited while loading the application.
Unrecognized option: --add-exports=java.base/jdk.internal.ref=ALL-UNNAMED
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
JVM Restarts disabled.  Shutting down.
<-- Wrapper Stopped
```

原因：需要 `java11`
[Supported Platforms](https://docs.sonarqube.org/latest/requirements/requirements/)

下载 `java11` 重新修改路径启动即可
