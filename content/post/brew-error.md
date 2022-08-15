---
# layout: post
title: "Brew: Version value must be a string; got a NilClass"
subtitle: ""
date: 2021-09-07
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "Version value must be a string; got a NilClass () (TypeError)"
---


# MacOS 11.1 Version value must be a string; got a NilClass () (TypeError)

好久没用brew，今天发现直接报错

![报错](http://cdn.chemputer.top/notebook/brew/error1.jpg)

**原因：** 发现跟我更新系统有关

![系统图](http://cdn.chemputer.top/notebook/brew/system.jpg)

**解决方案**：

同时更新brew

``` bash
brew update-reset
```