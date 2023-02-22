---
layout: post
title: "「React源码之二」 导读"
subtitle: "「源码阅读」第二层：掌握整体工作流程局部细节"
date: 2022-08-04
author: "汪洋龙"
categories: [React]
image: "img/react-bg.jpg"
---

> 1. 对于源码的阅读一开始一定要不求甚解
> 2. 查看源码剔除细节上的判断语句

## 重大版本更新

一开始不妨介绍下 `React` 的几个重大的节点。正所谓了解一个框架就要了解它的历史 👀 [React CHANGELOG](https://github.com/facebook/react/blob/main/CHANGELOG.md) 👈

### 18.0.0 (March 29, 2022)

默认开启 `concurrent` 模式

### 16.8.0 (February 6, 2019)

引入 `Hooks`

### 16.0.0 (September 26, 2017)

引入 `Fiber`

既然是重大更新，对于源码的学习自然绕不开。后续会对 `Fiber`、`Hooks`、`Concurrent` 模式都会有介绍。


## 整体架构

`React16` 之后的架构分为下面 `3` 层。

- Scheduler （调度器）
- Reconciler （协调器）
- Renderer  （渲染器）

`Reconciler` 上承 `Scheduler`，下接 `Renderer`，并且包含灵魂核心的 `Fiber`，所以需要优先介绍。

【前置知识】

- JSX
- Fiber

【正文】

- Reconciler
- Renderer
- Scheduler


