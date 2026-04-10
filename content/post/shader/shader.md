---
layout: post
title: "Shader"
subtitle: ""
date: 2023-03-03
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: ""
tags:
  - "GLSL"
  - "Shader"
---

# GLSL (GL Shader Language)

## 工具

### [KodeLife](https://hexler.net/kodelife)

强大的实时编辑预览工具

![kodelife](/post/shader/images/kodelife.jpg)

Tab 说明

- Vertex：顶点着色器
- Fragment：片段着色器
- Tess.Control （Tessellation Control Shader）： 曲面细分控制着色器
- Tess.Eval（Tessellation Evaluation Shader）： 曲面细分计算着色器
- Geometry（Geometry Shader）：几何着色器

### [shadertoy](https://www.shadertoy.com/)

非常有名的 Shader 学习网站，上面有着绚丽的 Shader 效果，并且有源码供学习

## 简介

- WebGL 在幕后使用 OpenGL ES。OpenGL 是一个用于处理 2D 和 3D 图形的快速库
- OpenGL ES 是 OpenGL 的一个版本，面向可能具有更多受限硬件的设备
