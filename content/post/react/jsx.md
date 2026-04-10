---
layout: post
title: "「React源码之三」【前置知识一】JSX"
subtitle: "「源码阅读」第二层：掌握整体工作流程局部细节"
date: 2022-08-05
author: "汪洋龙"
categories: [React]
image: "img/react-bg.jpg"
description: "JSX如何转化为FiberNode"
tags:
  - "react"
---

在准备步入 `Reconciler` 阶段之前，先了解下 `JSX` 将会如何被编译，并且编译之后的对象会有哪些属性。

## JSX to Fiber

在 `Babel` 官网中，下面这个例子

```js
function hello() {
  return (
    <div id="hello">
      <h1 className="title">This is Header</h1>
      <span>This is Content</span>
      <a>link</a>
    </div>
  );
}
```

会被编译为：

```js
function hello() {
  return /*#__PURE__*/ React.createElement(
    "div",
    {
      id: "hello",
    },
    /*#__PURE__*/ React.createElement(
      "h1",
      {
        className: "title",
      },
      "This is Header"
    ),
    /*#__PURE__*/ React.createElement("span", null, "This is Content"),
    /*#__PURE__*/ React.createElement("a", null, "link")
  );
}
```

那么 `createElement` 又做了什么呢？

在 `react/packages/react/src/React.js` 中定义了：

```js
import {
  createElement as createElementProd,
  ...
} from './ReactElement';
...
const createElement = __DEV__ ? createElementWithValidation : createElementProd;
```

也就是说，在开发模式调用 `createElementWithValidation`，在生产中调用 `createElementProd`。

### createElementProd

存在 `3` 个参数

- `type`：即例子中的 `div`、`h1`、`span`、`a` 标签
- `config`：即 `{id: "hello"}` 等属性
- `children`：若单一节点则是 `This is Content`、`link`，若多个则转化为列表

```js
export function createElement(type, config, children) {
  let propName;

  // Reserved names are extracted
  const props = {};

  let key = null;
  let ref = null;
  let self = null;
  let source = null;

  if (config != null) {
    ...
    // Remaining properties are added to a new props object
    for (propName in config) {
      ...
        props[propName] = config[propName];
      ...
    }
  }

  // Children can be more than one argument, and those are transferred onto
  // the newly allocated props object.
  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    const childArray = Array(childrenLength);
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    ...
    props.children = childArray;
  }

  ...
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}
```

1. 第一步先对 `第二个` 参数 `config` 进行遍历，拿到解析好的属性
2. 第二步解析 `第三个` 参数 `children`。若是多参数，则直接 `childArray[i] = arguments[i + 2];` 解析。
3. 第三步调用 `ReactElement` 方法返回组装好的对象。

### ReactElement

```js
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // This tag allows us to uniquely identify this as a React Element
    $$typeof: REACT_ELEMENT_TYPE,

    // Built-in properties that belong on the element
    type: type,
    key: key,
    ref: ref,
    props: props,

    // Record the component responsible for creating this element.
    _owner: owner,
  };
    ...
  return element;
};
```

## 扩展阅读

### Babel 如何转化 JSX

先引用一篇文章 [全新的 JSX 转换](https://github.com/reactjs/rfcs/blob/createlement-rfc/text/0000-create-element-changes.md#motivation) 👈 （意译）

### 动机

`React0.12` 前后，我们对 `key`， `ref`， `defaultProps` 做了一些小改动。特别是，它们很早在调用 `React.createElement(...)` 的时候就已经解决了。一切都是类组件时，就很合理。但从那以后，我们介绍了函数组件。`Hooks` 也让函数组件更为流行。是时候重新评估一些设计去做一些简化的事儿了（至少对函数组件来说）

元素的创建很频繁，因为被大量使用并且在渲染的时候频繁创建。

`React.createElement(...)` 从来没有打算成为 `JSX` 的实现。但它是当时我们的最佳实践。它目的是你可以手动编写 (如果你不想使用 `createFactory` 表单)。替代品并没有提供足够的价值可以保证它们用在任何地方。这有很多问题：

- 如果组件在每个元素创建调用期间都有 `.defaultProps`，我们需要对它进行动态测试。
-
-
-
-
-
-
-

### 设计细节

设计分为 3 步：1. 全新的 `JSX` 转换，2. 弃用和警告 3. 实际语意断言

#### 1. JSX 转换的不同

为了支持 React JSX 的变化，有很多编译、打包、下游工具等一系列组合需要升级。

###### 2.1 自动导入

首先我们需要避免在作用域中引入 `React`
理想情况下，元素的创建应该是编译器运行时的一部分。有一些实际方便的考虑。首先，我们有 `DEV` 和 `PROD` 两种模式。`DEV` 模式版本要复杂得多，并且集成到 `React` 中。我们还在版本之间进行了细微的更改——比如这个。

相比较更新编译器的工作链来说，部署 `npm` 包来迭代新版本会更容易点。因此，把具体的实现放在 `react` 包中是最好的。

理想情况下，使用 `JSX` 时无需任何导入：

```js
function Foo() {
  return <div /;
}
```

然后编译的时候就包含这个依赖，随后，打包器就能把它变成想要的。

```js
import {jsx} from "react";
function Foo() {
  return jsx('div', ...);
}
```

问题是并不是所有的工具都支持从一个转换新增一个依赖。第一步是搞清楚，在当前生态中它是如何做到的。

###### 2.2 把 `key` 从 `props` 中剥离出来

目前来说，`key` 作为 `props` 的一部分传递的，但未来我们想特殊处理。所以需要把它作为一个单独的参数传递。

```js
jsx("div", props, key);
```

###### 2.3 总是把 `children` 作为 `props` 传递

在 `createElement`，`children` 作为参数变量传递。在新的转换中，我们总是把它添加到 `props` 对象中。

我们用参数变量传值的原因是想在 `DEV` 模式中区分静态和动态 `children`。我提议是把 `<div{a}{b}</div` 编译成 `jsxs('div', {children: [a, b]})`，`<div{a}</div` 编译成 `jsx('div', {children:a})`。`jsxs` 函数表示上面的数组是 `React` 创建的。这个策略好的一点就是，即使你没有在 `PROD` 和 `DEV` 分离构建的步骤，我们仍然可以在 `PROD` 环境中发出关键警告，无需任何开销。

###### 2.4 DEV only transforms

我们对于 `DEV` 有特别的转化。`__source` 和 `__self` 并不是 `props` 的一部分。我们可以用分离开的参数传值。
一个可行方案是做分离函数去编译 `DEV`

```js
jsxDEV(type, props, key, isStaticChildren, source, self);
```

如果转换不匹配，我们很容易出错

###### 2.5 Spread only

这种特殊的匹配：

```js
<div {...props} />
```

可以安全的优化成：

```js
createElement("div", props);
```

是因为 `createElement()` 总是会克隆传递进来的对象。在新的转化中，我们想要避免在 `jsx()` 函数中克隆。多数情况下不会被观察到，因为 `JSX` 总是会创建一个新的内联对象。

或者
