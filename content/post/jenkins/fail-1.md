---
# layout: post
title: "React应用构建失败"
subtitle: ""
date: 2022-11-11
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "Treating warnings as errors because process.env.CI = true."
tags:
  - jenkins
---

# Jenkins 构建 React 应用失败


配置脚本：`Build` -> `Execute shell`

``` bash
#!/bin/sh
npm install && npm run build
```


报错信息

``` bash
...
> node scripts/build.js

Creating an optimized production build...

Treating warnings as errors because process.env.CI = true.
Most CI servers set it automatically.

Failed to compile.
...
```

原因：

[build-fails-on-warning-message](https://docs.netlify.com/configure-builds/troubleshooting-tips/#build-fails-on-warning-message)

```
Like many other CI tools and platforms, 
Netlify sets a build environment variable, 
CI=true, as a convention to indicate that your build is running in an automated environment. 
Many libraries use the presence of the CI variable to trigger changes in their behavior,
such as removing progress spinner animations or user prompts. 
In some cases, 
a library may also choose to treat warning messages as errors, failing the build.
```

大意是因为：**在某些情况下，有些库选择将警告消息视为错误，从而导致构建失败。**


解决方案：

配置脚本：`Build` -> `Execute shell`，新增 `CI=''` 或 `CI=false`

``` bash
#!/bin/sh
npm install && CI='' && npm run build

#!/bin/sh
npm install && CI=false && npm run build
```