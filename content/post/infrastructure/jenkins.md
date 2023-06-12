---
# layout: post
title: "前端基建之（五）自动化部署：Jenkins 安装和配置"
subtitle: ""
date: 2022-12-17
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "有时候部署的多了真的就不想手动部署了"
tags:
  - "infrastructure"
---

部署方案：`Docker`

## Jenkins 有哪些功能

- 定时拉取代码并编译（开发）
- 静态代码分析（开发）
- 定时打包发布测试版（开发，测试）
- 自定义额外的操作，如跑单元测试等（开发）
- 出错提醒（开发，测试）

## 第一步：安装部署 Jenkins

### 1. 拉取镜像 `jenkinsci/blueocean`

```bash
docker pull jenkinsci/blueocean
```

> 可以配置一下镜像源

```
"registry-mirrors": [
  "https://lkqzb0m7.mirror.aliyuncs.com"
]
```

### 2. 运行容器

```bash
docker run \
-d \
-p 10001:8080 \
-u root \
--restart=always \
-v /root/jenkins-volumes:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock \
jenkinsci/blueocean
```

---

```bash
# macOS
docker run \
-d \
-p 10001:8080 \
-u root \
-v /Users/chemputer/jenkinsocean-volumes:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock \
-v $(which docker):/usr/bin/docker \
jenkinsci/blueocean
```

---

```yml
# 待修改的jenkins的dockerfile
version: "3"
services:
  jenkins-compose:
    image: jenkins
    user: root # 表示在容器中以 root 用户运行
    ports:
      - "8088:8080"
      - "50000:50000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock # 表示 Docker 守护进程监听的 Unix 套接字。要在 Jenkins 容器中使用 docker 命令，则此选项是必需的。
      - /home/demo/jenkins-compose:/var/jenkins_home
```

`WARNING` 小插曲（运行之前需要执行）

> 需要修改下目录权限, 因为当映射本地数据卷时，`/home/docker/jenkins` 目录的拥有者为 `root` 用户，而容器中 `jenkins user` 的 `uid` 为 `1000`
>
> ```bash
> chown -R 1000:1000 /root/jenkins
> ```

### 3. 配置 `nginx.conf`

```bash
server{
        listen 80;
        server_name tool.chemputer.top;
        location /jenkins {
                proxy_pass http://127.0.0.1:10001;
                proxy_set_header Host $http_host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
        }
    listen 443 ssl;
    ssl_certificate /etc/nginx/cert/chemputer.com.pem;
    ssl_certificate_key /etc/nginx/cert/chemputer.com.key;
}
```

> 注意第`4`行的 `location /jenkins`

### 4. 访问页面

如果正常配置通过 `http://tool.chemputer.top` 是可以访问成功的，但是自己想通过 `nginx` 进行域名分类，所以想在 `nginx` 挂载 `/jenkins` 路径。 但是随后发现访问失败 `404`，进而排查错误。

### 5. 在服务器访问目标端口

```bash
curl 127.0.0.1:10001
```

```html
<html>
  <head>
    <meta http-equiv="refresh" content="1;url=/login?from=%2F" />
    <script>
      window.location.replace("/login?from=%2F");
    </script>
  </head>
  <body style="background-color:white; color:white;">
    Authentication required
    <!--
-->
  </body>
</html>
```

发现 `jenkins` 会切分路由，强制将 `/jenkins` 替换掉。需要在 `启动 jenkinsci/blueocean 时设置环境变量并修改nginx配置`

### 6. 设置环境变量

```bash
docker run \
-d \
-p 10003:8080 \
--restart=always \
-e TZ="Asia/Shanghai" \
-e JENKINS_OPTS="--prefix=/jenkins" \
-e JENKINS_ARGS="--prefix=/jenkins" \
-v /root/jenkins-volumes:/var/jenkins-volumes \
-v /var/run/docker.sock:/var/run/docker.sock \
jenkinsci/blueocean
```

> 注意 `6、7` 行的
>
> `-e JENKINS_OPTS="--prefix=/jenkins"`
>
> `-e JENKINS_ARGS="--prefix=/jenkins"`

### 7. 修改 `nginx` 配置

```bash
server{
        listen 80;
        server_name tool.chemputer.top;
        location /jenkins {
                proxy_pass http://127.0.0.1:10001/jenkins;
                proxy_set_header Host $http_host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
        }
    listen 443 ssl;
    ssl_certificate /etc/nginx/cert/chemputer.com.pem;
    ssl_certificate_key /etc/nginx/cert/chemputer.com.key;
}
```

> 注意 `4、5` 行的
>
> `location /jenkins {`
>
> `proxy_pass http://127.0.0.1:10001/jenkins;`

### 8. 重启

```bash
nginx -s reload
```

## 第二步：配置 Jenkins 打包

前端打包工具普遍使用 `node`，需要现在 `Jenkins` 中安装 `nodejs` 插件

### 1. 安装 `nodejs` 插件然后重启

![安装nodejs插件](http://cdn.chemputer.top/notebook/jenkins/step1.jpg)

### 2. 配置 `jenkins` `nodejs` 环境

![配置环境](http://cdn.chemputer.top/notebook/jenkins/step2.jpg)

### 3. 创建任务

![创建任务](http://cdn.chemputer.top/notebook/jenkins/step3.jpg)

### 4. 添加凭据

为了实现打包，`Jenkins` 需要能访问你的仓库，将代码放置在它的工作区。路径在 `root/jenkins-volumes/workspace`下。所以需要添加凭据

![添加凭据](http://cdn.chemputer.top/notebook/jenkins/step4.jpg)

### 5. 关联远程仓库

![关联远程仓库](http://cdn.chemputer.top/notebook/jenkins/step5.jpg)

### 6. 选择构建环境

![构建环境](http://cdn.chemputer.top/notebook/jenkins/step6.jpg)

### 7. 添加构建执行 shell

```bash
npm install && npm run build && cd dist && tar -zcvf dist.tar.gz *
```

![构建环境](http://cdn.chemputer.top/notebook/jenkins/step7.jpg)

### 8. 构建

执行一次 `立即构建`，会在工作区中找到工程文件，并会发现一个 `dist` 文件夹

![构建](http://cdn.chemputer.top/notebook/jenkins/step8.jpg)

## 第三步：部署至服务器

主要通过 `Publish over SSH` 插件实现部署

### 1. 安装 `Publish over SSH` 插件

### 2. 配置 `Publish over SSH` 项

![Publish over SSH](http://cdn.chemputer.top/notebook/jenkins/step9.jpg)

`WARNING` 小插曲

> 配置秘钥之后，点击 `Test Configuration` 会发现 `Jenkins` 报错，`jenkins.plugins.publish_over.BapPublisherException: Failed to add SSH key. Message [invalid privatekey: [B@3ed3d4f0]`

**原因：**
生成密钥的 `openssh` 的版本过高的原因, 会发现秘钥文件是以 `-----BEGIN OPENSSH PRIVATE KEY-----` 开头，而目前 `Jenkins 2.303.2` 暂不支持

### 3. 生成指定 `ssh` 秘钥格式

```bash
ssh-keygen -m PEM -t rsa -b 4096

# -m 指定秘钥格式，PEM 是 RSA 之前的旧格式
# -b 指定秘钥长度，对于 RSA 秘钥，最小要求 768 位，默认是 2048 位
```

```bash
cat ~/.ssh/id_rsa_pem
```

秘钥文件以 `-----BEGIN RSA PRIVATE KEY-----` 开头，`Jenkins` 能够识别通过。

4. 将生成的 `私钥` 填写在 `Publish over SSH` 的 `key` 中，将 `公钥` 填写在目标服务器的 `authorized_keys` 中，执行 `Test Configuration` ，`Success` 成功！

5. 配置构建后操作选择 **Send build artifacts over SSH**

- Name:

`系统管理>系统设置` 设置的 `SSH Sverver` 的名字列表。

- Source files: `dist/dist.tar.gz`

复制到运程机上的文件，相对 `Jenkins workspace` 的路径，也支持表达式。

- Remove prefix: `dist`

文件复制时要过滤的目录，这样 `dist` 文件夹就不会复制。

- Remote directory:

文件得到到远程机上的目录，此目录是相对于 `SSH Server` 中的 `Remote directory` 的，如果不存在将会自动创建。

- Exec command: `cd /p/a/t/h && tar -zxvf dist.tar.gz && rm -rf dist.tar.gz`

在这里可以填写在运程机器上执行的脚本

## 第四步：git push 代码执行自动构建

`待完成...`

## 扩展阅读：一些自动化工具

- `Selenium`
- `jenkins`
- `ansible`
- `puppet`
- `nagios`
- `chef`
