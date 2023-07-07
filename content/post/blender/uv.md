---
# layout: post
title: "Blender（三）UV"
subtitle: ""
date: 2023-07-05
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "关于贴图"
tags:
  - "Blender"
---

## 1. 是什么

3D 物体的 2D 展示

> `XYZ` 已经用于三维空间的坐标轴，`UV` 则用来表示纹理贴图上的坐标轴

## 2. 应用场景

建模时不可避免的对物体进行各种形变和拉伸，贴图之后会很奇怪。这是用 `uvchecker` 贴图之后的效果，需要处理之后让棋盘格显示为正方形。

![uvchecker](/post/blender/images/uvchecker.jpg)

## 3. 处理步骤

### 3.1 进入 UV 编辑

- 上方的 `布局` 和 `UV编辑` 是相互独立的，所以在其中一个的设定在另一个并不生效
- 在 `物体模式` 中选中，进入 `编辑模式`，按 `A` 键全选，左侧就有相应的映射

![uvedit](/post/blender/images/uvedit.jpg)

### 3.2 处理 UV

- 有多个面的 UV 可以通过 `UV` 的 `拼排孤岛` 分开
- 再通过 `选择` 的 `选择相连元素` 进行调整

![uvlink](/post/blender/images/uvlink.jpg)
![uvisland](/post/blender/images/uvisland.jpg)

### 3.3 着色网络

- 进入上方的 `着色` 面板，关注下方的 `着色编辑器`，即对应 `Texture` + `Shader` = `Material`

![shader](/post/blender/images/shader.jpg)

- 映射节点需要与 UV 交互
