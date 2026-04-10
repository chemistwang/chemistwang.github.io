---
# layout: post
title: "「机器学习之二」用 TensorFlow.js 进行机器学习"
subtitle: ""
date: 2023-03-23
author: "汪洋龙"
categories: [Tech]
image: "img/ai.jpg"
description: ""
---

# 用 TensorFlow.js 进行机器学习

## 1. 简介

> [TensorFlowJS 官网](https://www.tensorflow.org/js)
>
> [Google 中国：基于 TensorFlow.js 的下一代 Web 应用](https://www.bilibili.com/video/BV1Kq4y1k7dw)

一个使用 `JavaScript` 编写的机器学习库。具有 `低延迟`、`高私密性`、`低部署成本` 等优点。还可用于 `Node.js` 支持的服务器和 `IoT` 设备环境。

### 1.1 使用方法

- 运行现有模型：预封装的 `JavaScript` 模型或由 `Python` 或 `TF Lite` 转换而来的模型
- 重新训练现有模型： 利用迁移学习
- 使用 JS 编写模型：从头开始训练

### 1.2 预制模型

> [预制模型](https://www.tensorflow.org/js/models)

- 视觉：图像分类、对象监测
- 人体：人体分割、姿态估计、手部姿态估计
- 文本：恶意文本、句子编码、BERT 问答、对话意图监测
- 声音：语音指令识别
- 其他：KNN 分类器

### 1.3 应用场景

- `inSpace`：客户端恶意文本过滤器
- `includeHealth`: 使用姿态检测模型判断用户是否按规定完成训练动作
- `Tone Transfer`: 将您的声音转化成乐器演奏声
- `Project Shuwa`: 手语识别，让世界更加没有障碍

### 1.4 2021 年的新模型

新的姿态检测模型

- `MoveNet`
- `SinglePose`
- `MultiPose`
- `BlazePose` 2D 和 3D
