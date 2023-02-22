---
layout: post
title: "JS体系之六【DOM事件】"
subtitle: ""
date: 2022-09-01
author: "汪洋龙"
categories: []
image: "img/coffee-bg.jpg"
description: "很重要"
---


```html
<div onclick="console.log('div')">
  <p onclick="console.log('p')">
    Click here!
  </p>
</div>
```

```
p
div
```

在事件传播过程中，有3个阶段：捕获、目标和冒泡。

默认情况下，事件处理程序在冒泡阶段执行（除非将 useCapture 设置为 true），它从最深的嵌套元素向外。