---
# layout: post
title: "《Machine Learning - Andrew Ng》学习笔记（二）"
subtitle: ""
date: 2023-03-31
author: "汪洋龙"
categories: [Tech]
image: "img/ai.jpg"
description: "高级学习算法（Advanced Learning Algorithms）"
mathjax: true
tags:
  - ai
---

课程目标

- 了解神经网络（深度学习算法）
- 决策树
- 如何构建机器学习系统

# 第一周：高级学习算法

## 1. 神经网络

### 1.1 神经元和大脑

![神经元和大脑](/post/ml/AndrewNg/images/c2-nn.jpg)

### 1.2 需求预测

![需求预测](/post/ml/AndrewNg/images/c2-nna.jpg)

### 1.3 应用于图像感知

## 2. 神经网络模型

### 2.1 神经网络层

### 2.2 更复杂的神经网络

### 2.3 推断：预测（向前传播）

## 3. TensorFlow 实现

### 3.1 用代码推断

[安装](https://www.tensorflow.org/install/pip#macos)

```bash
# There is currently no official GPU support for MacOS.
python3 -m pip install tensorflow
# Verify install:
python3 -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
```

![TensorFlow 实现](/post/ml/AndrewNg/images/c2-tfcode1.jpg)

![TensorFlow 实现](/post/ml/AndrewNg/images/c2-tfcode2.jpg)

### 3.2 TensorFlow 中的数据

### 3.3 搭建一个神经网络

### 3.4 单个网络层上的向前传播

### 3.5 向前传播的一般实现

## 4. AGI（artificial general intelligence）的推测

![AGI](/post/ml/AndrewNg/images/c2-agi.jpg)

## 5. 向量化（可选）

### 5.1 为何神经网络如此高效

![loop vs vec](/post/ml/AndrewNg/images/c2-loopvsvec.jpg)

### 5.2 矩阵乘法

### 5.3 矩阵乘法规则

### 5.4 矩阵乘法代码

# 第二周：神经网络训练

## 1. TensorFlow 实现

## 2. 训练细节

## 3. 多分类问题

### 3.1 Softmax
