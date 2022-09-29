---
# layout: post
title: "TS2559: Type '{ children: never[]; }' has no properties in common with type 'IntrinsicAttributes'."
subtitle: ""
date: 2022-09-29
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "TS"
tags:
  - ts
---

封装好的组件准备在外层进行调用：

```jsx
interface Props {
    title: string
}

function MyComponent(props: Props) {
    const { title } = props;
    return <h1>{title}</h1>
}

export default MyComponent;
```

结果 `TS` 直接报错


```
ERROR in src/App.tsx:86:6
TS2322: Type '{ children: never[]; title: string; }' is not assignable to type 'IntrinsicAttributes & Props'.
  Property 'children' does not exist on type 'IntrinsicAttributes & Props'.
```

``` tsx
  ...
  return <>
    <MyComponent title="App Title"> 👈
    ~~~~~~~~~~~~
    </MyComponent>
  </>
```

寻思子组件中并没有 `children` 属性，但是如果这么写就没有问题。原因竟然是因为我换行了

```tsx
  ...
  return <>
    <MyComponent title="App Title"></MyComponent>
  </>
```

问题解析：

> 写在标签内部的任何代码，该组件均被视为存在 `children` 属性，即使是空格。（也能理解，属于空字符串）

解决方案：

1. `props` 用 `any` 定义

之前用 `any` 去定义类型，这种错误就很容易被忽略。

```tsx
function MyComponent(props: any) { // 👈 很容易养成 anyScript 的习惯
    const { title } = props;
    return <h1>{title}</h1>
}

export default MyComponent;
```

所以不推荐

2. 用 `React.ReactNode` 定义 `children`

```jsx
interface Props {
    title: string
    children: React.ReactNode; // 👈 定义类型
}

function MyComponent(props: Props) {
    const { title, children } = props;
    return <>
        <h1>{title}</h1>
        <div>{children}</div>
    </>
}

export default MyComponent;
```

调用方法：

``` tsx
  ...
  return <>
    <MyComponent title="App Title">
        <div>Something</div>
    </MyComponent>
  </>
```