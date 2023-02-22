---
# layout: post
title: "BIM 及其应用（科普篇）"
subtitle: ""
date: 2022-06-23
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "BIM应用科普"
---

# BIM 应用

## BIM + 模型轻量化

- 应用阶段轻量化

	- `Revit` 模型 -> 网页 （ `Revit` 模型是不能直接放上去的，需要格式转化）
	- 转化工具：`Gltf` ｜ `3D Tiles`

- 如何轻量化
	
	- BIM平台
		- `Revit`
		- `Catia`
		- `Bentley`

	- 业务场景使用平台
		- Web：  `Gltf` ｜ `3D Tiles`
		- 手机平台：  同上
		- 游戏引擎 `fbx`
		- 建筑行业通用格式 `IFC`

	- 转换3个阶段
		- BIM -> 3D 
			1. 原生
				- 优点：软件自带导出
				- 缺点：没有材质，没有属性，很多格式不支持
			2. 插件开发
				- Revit：导出接口 C#
				- Catia：导出接口 C++
				- Bentley：导出接口 C#
				- 导出时，参数控制导出几何精细程度实现轻量化

		- 3D -> 轻量化3D
			1. fbx
				- Apose
				- Decimate in Blender
			2. gltf
				- google draco
			3. 3dtiles
				- google draco
			4. IFC
				- solibri-ifc-optimizer

			关键词 `compression`

		- 文本数据轻量化 压缩


	- 成熟厂商
		- [`广联达`](https://bimface.com/docs-center)  👈
		- [`大象云`](https://open.daxiangyun.com/home)  👈
		- [`Forge`](https://viewer.autodesk.com/)  👈


## BIM + 数模分离

数据 - 属性
模型 - 几何

- 1. 类别
	- 数模分离
	- 数模不分离 (数据库)

- 2. 什么时候需要数模分离
	- 特定格式
		- 不分离 `IFC`
		- 分离 `FBX`
		- 可分可不分 `GLTF`
	- 属性经常变化
		- 属性在模型中，属性改变，模型需要跟着改变，费时
	- 属性来自其他数据源
		- 构件上绑定的图纸信息，工程量信息
	- 模型展示
		- 数据分离之后模型轻量化

- 3. 实现数模分离
	- 数据和模型通过elementId对应

- 4. 注意事项
	- 在大场景中elementId不够用
		- 多个模型在同一个场景 （只有6位）
		- 场景模型来自不同平台 （不统一）
		- 采用GUID，区分为一构件



# BIM + Web开发

- 1. 技术选型：webGL

- 2. 优点
 	- 快速查看BIM模型
 	- 施工建设管理系统
 	- 运维管理系统
 	- 在线协同设计

- 3. 实现方案
	- BIM厂家
		- Autodesk -> Forge `https://viewer.autodesk.com/`
		- Bentely -> itwin.js `https://www.itwinjs.org/`

	- 自研（webGL太底层，不方便使用）
		- three.js **小场景， 开源**
		- babylon.js **小场景， 开源**
		- cesium.js **大场景， 开源**

- 4. 常见问题探讨
	- 模型上云发生什么
	 	- 模型格式解析
	 		- 解析成什么
	 			- 现有3D格式 `gltf | 3Dtiles`
	 			- 自研3D格式 `bentley imodel`
	 		- 如何解析（以revit模型举例子）
	 			- 开启建模软件 
	 			- 不开启建模软件
	 				(不开启Revit情况下读取修改Revit数据)
	 				`https://www.opendesign.com/cn/products/bimrv`

	 	- 加载网页中

	- 在线设计现状
		- 国外 onshape
		- 国内 酷家乐 ｜ 云图三维

	- 大规模场景如何快速加载
		- 平台上 （选GIS平台，比如cesium/arcgis等）

		- 轻量化 （BIM模型、倾斜摄影模型）

		- 加载手段（LOD加载、流式加载（分批加载）、缓存加载）



## BIM + 计算机图形学

- 1. 什么是图形学
	如何显示问题

- 2. 为什么要了解图形学
	- 专业词汇：可编程渲染管线、显卡光线追踪技术
	- 解决问题：BIM应用阶段
		- 为什么后面物体会被挡住 **画家算法**

		- 为什么经常用三角网面 **任何面都可以用三角形来表示**

		- 鼠标如何找到屏幕上的物体 **射线法**

		- 模型显示如何接近真实 **渲染手法**

		- 模型在网页显示为什么出现锯齿 **抗锯齿**


- 3. BIM建模平台与图形学
	- 略


## BIM + 人工智能

- AI主流算法

	- 机器学习
	- 深度学习
		- CNN 卷积神经网络 ｜ 图像分类、目标检测、语义分割
		- RNN 循环神经网络 ｜ 语音识别、机器翻译
		- GANs 生成对抗网络


## BIM + 有限单元法

## BIM + GIS

## BIM + Web

## BIM + 手机

## BIM + Unity 3D

## BIM + UE4

## BIM + 数值计算