---
# layout: post
title: "《PyTorch 深度学习实践》学习笔记"
subtitle: ""
date: 2023-03-24
author: "汪洋龙"
categories: [Tech]
image: "img/ai.jpg"
description: "By 刘二大人"
mathjax: true
tags:
  - ai
---

# 《PyTorch 深度学习实践》学习笔记

非常庆幸的是，在我准备接触机器学习框架的时候，`PyTorch` 在前不久推出了它的 `2.0` 版本。

> [《PyTorch 深度学习实践》完结合集 - 刘二大人](https://www.bilibili.com/video/BV1Y7411d7Ys)

## 第一讲 Overview

### 常用的算法思维（人工的设计）

- 穷举法
- 贪心算法
- 分治法（快排）
- 动态规划

`维度诅咒`

### Rule-based system VS Representation learning

![Rule-based system VS Representation learning](/post/ml/pytorch/images/2.jpg)

- 基于规则：
- 表示学习：

### 神经网络的历史

`perceptron`

- 算法
- 数据集
- 算力

## 第二讲 线性模型

### 1. 步骤

- 准备数据集（DataSet）
- 模型的选择/设计（Model）
- 训练（Training）
- 推理（Inferring）

### x. 代码实现

```py
import numpy as np
import matplotlib.pyplot as plt

# 1. 数据集
x_data = [1.0, 2.0, 3.0]
y_data = [2.0, 4.0, 6.0]

# 2. 定义模型
def forward(x):
    return x * w

# 3. 定义 loss 函数
def loss(x, y):
    y_pred = forward(x)
    return (y_pred - y) * (y_pred - y)

w_list = []
mse_list = []

# 4. 训练（间隔0.1）
for w in np.arange(0.0, 4.1, 0.1):
    print('w=', w)
    l_sum = 0
    for x_val, y_val in zip(x_data, y_data):
        y_pred_val = forward(x_val)
        loss_val = loss(x_val, y_val)
        l_sum += loss_val
        print('\t', x_val, y_val, y_pred_val, loss_val)
    print('MSE=', l_sum/3)
    w_list.append(w)
    mse_list.append(l_sum/3)

# 5. 绘图
plt.plot(w_list, mse_list)
plt.ylabel('Loss')
plt.xlabel('w')
plt.show()
```

执行 `python index.py` 会生成下图

![Loss-w](/post/ml/images/3.jpg)

在深度学习过程中，一般会用 `Epoch(轮数)` 作为 `x` 轴。在训练过程中，可以选择用 [Visdom](https://github.com/fossasia/visdom) 来进行可视化的实时观察。

### x. 思考作业

用线性模型 `$\hat{y} =x\  \times w\  +\  b$` 画出 cost 图。

> Tips: 绘制 3d 图像用 `np.meshgrid()` 函数非常流行。

---

## 第三讲 梯度下降算法

上述其实是一个`穷举法`。这种算法在低纬度还相对可控，但在高纬度，多参数的情况下，计算量会指数级暴增。

### 1. 优化思路

![优化问题](/post/ml/pytorch/images/4.jpg)

所以找到 `cost 函数` 的最小值引申为它的优化问题。

![核心思想](/post/ml/pytorch/images/5.jpg)

![核心思想](/post/ml/pytorch/images/6.jpg)

### 2. 代码实现

#### 2.1 准备数据集

```py
x_data = [1.0, 2.0, 3.0]
y_data = [2.0, 4.0, 6.0]
```

#### 2.2 设定一个权重（weight）的初始值

```py
w = 1.0
```

#### 2.3 根据 `$\hat{y} =x\  \times w\  +\  b$` 定义模型

```py
def forward(x):
    return x * w
```

#### 2.4 根据 `$cost(w)=\frac{1}{N} \sum_{n=1}^{1} (\hat{y}-y_{n})^{2} $` 定义 cost 函数

```py
def cost(xs, ys):
    cost = 0
    for x,y in zip(xs, ys):
        y_pred = forward(x)
        cost += (y_pred - y) ** 2
    return cost / len(xs)
```

#### 2.5 根据 `$\frac{\partial cost}{\partial w}=\frac{1}{N}\sum_{n=1}^{N}2*x_{n}*(x_{n}*w-y_{n})$` 定义梯度函数

```py
def gradient(xs, ys):
    grad = 0
    for x, y in zip(xs, ys):
        grad += 2 * x * (x * w - y)
    return grad / len(xs)
```

#### 2.6 根据 `$w=w-\alpha \frac{\partial cost}{\partial w}$` 开始训练

```py
print('Predict (before training)', 4, forward(4))
for epoch in range(100):
    cost_val = cost(x_data, y_data)
    grad_val = gradient(x_data, y_data)
    w -= 0.01 * grad_val
    print('Epoch:', epoch, 'w=', w, 'loss=', cost_val)
    print('Predict (after training)', 4, forward(4))
```

### 3. 随机梯度下降

### 4. 折中的方法 Batch

批量的随机梯度下降 Mini-Batch

## 第四讲 反向传播(Back Propagation)

## 第五讲 Pytorch 实现线性回归

## 第六讲 Logistic 回归

## 第七讲 处理多维特征的输入

## 第八讲 加载数据集

## 第九讲 多分类问题

## 第十讲 CNN 基础

## 第十一讲 CNN 进阶

## 第十二讲 RNN 基础

## 第十三讲 RNN 进阶
