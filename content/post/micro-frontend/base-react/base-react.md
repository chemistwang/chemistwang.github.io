---
layout: post
title: "微前端【qiankun】小试（二）添加主应用和子应用视图"
subtitle: ""
date: 2023-01-03 17:00:00
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "分久必合，合久必分"
tags:
  - qiankun
---

上一篇似乎一个初步的 demo 就已经跑起来了，在实践中我遇到过单点应用跳转多平台的需求，不妨趁机试着做成一个完整的 demo。

# 添加主应用和子应用视图

## 1. 添加主应用逻辑

- 基于 `antd5.x` 实现 UI
- 基于 `react-router6.x` 实现路由

### 1.1 路由设定

| 路由                             | 应用                              |
| -------------------------------- | --------------------------------- |
| /login                           | 登陆                              |
| /workbench                       | 工作台                            |
| /workbench/apps                  | 工作台-应用管理                   |
| /workbench/system/users          | 工作台-系统设置-用户管理          |
| /workbench/system/roles          | 工作台-系统设置-角色管理          |
| /workbench/mini-react18          | 子应用 mini-react18               |
| /workbench/mini-react18/form     | 子应用 mini-react18 form 菜单     |
| /workbench/mini-react18/timeline | 子应用 mini-react18 timeline 菜单 |
| /workbench/mini-vue2             | 子应用 mini-vue2                  |

### 1.2 路由修改

新建的 `router/index.tsx`，注意下子应用的容器和对应的路由。

| 子应用容器                      | 路由             |
| ------------------------------- | ---------------- |
| `<div id="mini-react18"></div>` | `mini-react18/*` |
| `<div id="mini-vue2"></div>`    | `mini-vue2/*`    |

```js
...
const router = createBrowserRouter([
  {
    path: "/",
    element: <LoginPage></LoginPage>,
  },
  {
    path: "/workbench",
    element: <WorkbenchPage></WorkbenchPage>,
    children: [
      {
        path: "",
        element: <MainModule></MainModule>,
        children: [
          {
            path: "apps",
            element: <AppsModule></AppsModule>,
          },
          {
            path: "system/users",
            element: <UsersModule></UsersModule>,
          },
          {
            path: "system/roles",
            element: <RolesModule></RolesModule>,
          },
        ],
      },
      {
        path: "mini-react18/*",
        element: <div id="mini-react18"></div>,
      },
      {
        path: "mini-vue2/*",
        element: <div id="mini-vue2"></div>,
      },
    ],
  },
]);
...
```

### 1.3 主应用挂载路由修改

因为路由变了，对应的 `activeRule` 也需要跟着变

```js
...
registerMicroApps([
  {
    name: "react app 18",
    entry: "//localhost:11111",
    container: "#mini-react18",
    activeRule: "/workbench/mini-react18",
  },
  {
    name: "vue app 2",
    entry: "//localhost:11112",
    container: "#mini-vue2",
    activeRule: "/workbench/mini-vue2",
  },
]);
...
```

## 2 添加 mini-react18 子应用逻辑

### 2.1 修改路由

新建的 `router/index.tsx`，需要设定 `basename`

```js
...
const router = createBrowserRouter(
  [
    {
      path: "/",
      element: <WorkbenchPage></WorkbenchPage>,
      children: [
        {
          path: "form",
          element: <FormPage></FormPage>,
        },
        {
          path: "timeline",
          element: <TimelinePage></TimelinePage>,
        },
      ],
    },
  ],
  {
    basename: window.__POWERED_BY_QIANKUN__ ? "/workbench/mini-react18" : "/",
  }
);
...
```

## 3. 添加 mini-vue2 子应用逻辑

目前暂无逻辑

## 4. 视图展示

可以注意下浏览器路由

登陆

![login](/post/micro-frontend/base-react/login.png)

主应用

![main](/post/micro-frontend/base-react/main.png)

子应用 `mini-react18`

![mini-react18](/post/micro-frontend/base-react/mini-react18.png)

子应用 `mini-vue2`

![mini-vue2](/post/micro-frontend/base-react/mini-vue2.png)
