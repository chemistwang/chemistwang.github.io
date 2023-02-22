---
# layout: post
title: "TS配置文件"
subtitle: "tsconfig.json"
date: 2022-11-07
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "TS"
tags:
  - ts
---

## strict 

[strict文档](https://www.typescriptlang.org/tsconfig#strict) `true` `false`


```
The strict flag enables a wide range of type checking behavior that results in stronger guarantees of program correctness. Turning this on is equivalent to enabling all of the strict mode family options, which are outlined below. You can then turn off individual strict mode family checks as needed.

Future versions of TypeScript may introduce additional stricter checking under this flag, so upgrades of TypeScript might result in new type errors in your program. When appropriate and possible, a corresponding flag will be added to disable that behavior.
```