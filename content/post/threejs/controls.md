---
layout: post
title: "ThreeJS（一）基础篇"
subtitle: ""
date: 2023-02-27
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: ""
tags:
  - "threejs"
---

# Controls

- orbitControls（轨道控制器）：三维场景进行缩放、平移、旋转操作以及自动旋转，运动轨迹是椭圆形
- TrackballControls（轨迹球）：------------------------------------，运动轨迹是无闭合
- Flycontrols（飞行控制器）：第一人称视角，模拟飞机操作杆，能产生侧飞的感觉
- FirstPersonControls（第一人称控制器）：第一人称视角，默认人的行走
- PointerLockControls（游戏控制器）：第一人称视角，能够实现前后左右行走、跳跃、碰撞
- MapControls（地图控制器）：类似地图模型或者不希望用户对模型进行反转的时候用这个控件，对远处的模型逐渐模糊
- 可视化平移控件（transformControls）: 对模型进行平移、放大缩小、拉伸挤压
- 拖拽控件 DragControls：对模型进行拖拽以及射线选中功能
- EditorContorls： 对模型进行拖拽
- OrithographicTrackballContorls
- DeviceOrientationcontrols(设备朝向控制器)：该控制器可以使得摄像机依据设备的朝向（手机陀螺仪）来进行调整
