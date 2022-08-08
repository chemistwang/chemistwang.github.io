---
layout: post
title: "Build Your Own React (译文)"
subtitle: "「源码阅读」第一层：掌握术语、基本实现思路"
date: 2021-09-01 17:58:34
author: "汪洋龙"
categories: [React]
image: "img/react-bg.jpg"
description: "从零到一构建自己的React"
---

> 参考资料:
> [build-your-own-react](https://pomb.us/build-your-own-react/)

# 构建自己的 React

**⚠️【注】因本人水平有限，部分语句使用意译。会根据水平提升及时修正完善🧐**

*Rodrigo Pombo November 13, 2019*

我们准备一步一步重写 `React`。下面的架构来自 `React` 源码，但把所有的优化和不必要的功能省略掉了。

如果你读过我之前的一篇帖子 [`build your own React`](https://engineering.hexacta.com/didact-learning-how-react-works-by-building-it-from-scratch-51007984e5c5)，跟上一篇不同之处在于这一篇基于 `React16.8`，我们可以使用 `hooks`并且抛弃掉类组件代码

你可以在 [`这里`](https://github.com/pomber/didact) 找到实现的代码仓库，在 [`这里`](https://www.youtube.com/watch?v=8Kc2REHdwnQ&feature=youtu.be&ab_channel=GrUSP) 看到相关视频

从零开始，我们会一步一步把这些加到我们的`React`中

- 第一步：`createElement` 函数
- 第二步：`render` 函数
- 第三步：`concurrent` 模式
- 第四步：`Fibers`
- 第五步：`Render` 和 `Commit` 阶段
- 第六步：`Reconcilation`
- 第七步：`Function` 组件
- 第八步：`Hooks`


## 前言

开始之前，我们回顾一些基本概念。如果你已经知道 `React`, `JSX` 和 `DOM` 元素如何运行，那么可以跳过。

我们用 `3` 行代码就可以启用一个 `React App`。

第一步: 定义一个 `React element`

第二步: 从 `DOM` 获取一个节点

第三步：在 `container` 中渲染 `React element`

```js
const element = <h1 title="foo">Hello</h1>
const container = document.getElementById("root")
ReactDOM.render(element, container)
```

**让我们移除所有的React代码，用原生的JavaScript代替**

---


第一行我们用 `JSX` 定义了元素，但这不是有效的 `Javascript`，所以为了用原生`JS`代替，我们需要把他变成有效的`JS`

`JSX` 是通过像 `Babel` 这样的构建工具转换成 `JS`。这种转化通常来说比较简单：调用 `createElement` 方法，把 `标签名`、`属性` 和它的 `子元素` 作为参数传入。

---

`React.createElement` 根据参数创建了一个对象。除了校验之外，这就是它做的全部了。所以可以安全替换掉。

```js 
const element = React.createElement(
  "h1", 
  { title: "foo" }, 
  "Hello"
);
// const container = document.getElementById("root")
// ReactDOM.render(element, container)
```

这就是一个元素真正的样子，一个对象包含两个属性 `type` 和 `props`（其实有[很多](https://github.com/facebook/react/blob/f4cc45ce962adc9f307690e1d5cfa28a288418eb/packages/react/src/ReactElement.js#L111)，我们只关心它俩）

```js
const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  },
};
// const container = document.getElementById("root")
// ReactDOM.render(element, container)
```

`type` 定义了我们想要创建什么类型的 `DOM` 节点，就是当你想要创建一个 `HTML` 元素的时候，传入 `document.createElement` 方法的 `标签名`。它也可以是一个函数，不过我们会在 [**第七步**](#第七步function组件) 说明。

`props` 是另一个对象，它包含了所有来自 `JSX` 的各项属性。它还有一个特殊的属性：`children`。

`children` 在当前这个例子里是个 `string`，但多数情况是一个包含多个元素的 `array`。这也是为什么说元素也是树 🌲。

---

另外一处需要替换的代码是 `ReactDOM.render`。

`render` 表示 `React` 作用在哪个 `DOM` 上，那我们用自己的方式实现更新。

---

首先我们用元素的 `type` 创建一个`节点`，当前例子就是 `h1`。

然后我们把所有属性放在这个 ` 节点` 里。当前只有 `title`。

> 为了避免混淆，我用 `element` 代表 `React` 元素，用 `node` 代表 `DOM` 元素。


```js
const element = {
  type: "h1",
  // props: {
    title: "foo",
    // children: "Hello",
  },
};
// const container = document.getElementById("root")

const node = document.createElement(element.type);
node["title"] = element.props.title;

// const text = document.createTextNode("");
// text["nodeValue"] = element.props.children;

// node.appendChild(text);
```

---

然后为子元素创建节点。我们只有一个 `string` 作为子元素，那么就创建一个 `text node`

为了在后续步骤中用同一种方法处理所有元素，我们用 `textNode` 而没用`innerText`。

注意一下，就像设置 `h1` 的 `title` 属性一样，我们用同样的方法设置 `nodeValue`，就像这个字符串有了属性一样 `props: {nodeValue: 'hello'}`

```js
const element = {
  // type: "h1",
  // props: {
  //   title: "foo",
    children: "Hello",
  },
};
// const container = document.getElementById("root")

// const node = document.createElement(element.type);
// node["title"] = element.props.title;

const text = document.createTextNode("");
text["nodeValue"] = element.props.children;

// node.appendChild(text);
```

---


最后面把 `textNode` 放在 `h1` 中，然后把 `h1` 放到 `container` 里面。

```js
// const element = {
//   type: "h1",
//   props: {
//     title: "foo",
//     children: "Hello",
//   },
// };
const container = document.getElementById("root")

// const node = document.createElement(element.type);
// node["title"] = element.props.title;

// const text = document.createTextNode("");
// text["nodeValue"] = element.props.children;

node.appendChild(text);
container.apendChild(node)
```

---

这下就和刚才一样了，区别是没有用 `React`

```js
const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  },
};
const container = document.getElementById("root")

const node = document.createElement(element.type);
node["title"] = element.props.title;

const text = document.createTextNode("");
text["nodeValue"] = element.props.children;

node.appendChild(text);
container.apendChild(node)
```

---

## 第一步：`createElement` 函数

我们从另一个 `app` 重新开始。这次用自己实现的 `React`。

先从实现 `createElement` 开始。

让我们把 `JSX` 转换为 `JS`，这样可以看到 `createElement`的调用。

```js
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
);
const container = document.getElementById("root");
ReactDOM.render(element, container);
```

---

正如`上一步`所看到的，一个 `element` 是一个有 `type` 和 `props` 的对象。我们的函数只需要做一件事：就是创建这个对象。

```js
const element = React.createElement(
  "div",
  { id: "foo" },
  React.createElement("a", null, "bar"),
  React.createElement("b")
);
// const container = document.getElementById("root");
// ReactDOM.render(element, container);
```

---

我们对 `props` 使用 `对象扩展运算符`，对 `children` 用 `rest参数语法`，这样的话 `children` 属性就会一直是一个 `array`


```js
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children,
    },
  };
}
```

例如，`createElement('div')` 返回

```js
{
  "type": "div",
  "props": { "children": [] }
}
```

`createElement("div", null, a)`返回

```js
{
  "type": "div",
  "props": { "children": [a] }
}
```

`createElement("div", null, a, b)`返回

```js
{
  "type": "div",
  "props": { "children": [a, b] }
}
```

---

`children` 数组可能也会包含 `string` 或者 `numbers` 这样的原始值，所以我们对这类不是对象的值创建一个特殊类型：`TEXT_ELEMENT`

> 当它们不是 `children`时，`React` 并不会包裹原始值或者创建一个空数组。这么做是因为可以简化代码，对于我们的实现库来说，简洁会更重要。

```js
// function createElement(type, props, ...children) {
//   return {
//     type,
//     props: {
//       ...props,
      children: children.map((child) =>
        typeof child === "object" 
        ? child
        : createTextElement(child)
      ),
//     },
//   };
// }

function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  };
}
```

---

我们仍然用了 `React` 的 `createElement`

为了替换的目的，我们给自己的实现库起一个名字。

我们需要一个听上去像 `React` 的名字，同时也能达到 `didactic` 目的。

```js
...
// const element = React.createElement(
//   "div",
//   { id: "foo" },
  React.createElement("a", null, "bar"),
  React.createElement("b")
// )
...
```

---

就叫 `Didact` 吧。

```js
const Didact = {
  createElement,
};
const element = Didact.createElement(
  // "div",
  // { id: "foo" },
  Didact.createElement("a", null, "bar"),
  Didact.createElement("b")
);
```

但我们还想在这用 `JSX` 语法，怎么告诉 `babel` 是要用 `Didact` 的 `createElement`呢？


---

如果我们有一个像这样的注释，当 `babel` 编译的时候就会用我们定义的函数。

```js
/** @jsx Didact.createElement */
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
);
```


## 第二步：`render`函数

接下来，我们需要实现一个自己的 `ReactDOM.render` 函数。

```js
...
ReactDOM.render(element, container);
...
```


---

现在只考虑如何添加，后面再去实现更新和删除。

```js
function render(element, container) {
  // TODO create dom nodes
}

const Didact = {
  // createElement,
  render
}

/** @jsx Didact.createElement */
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
);

// const container = document.getElementById('root');
Didact.render(element, container);
```

---

用 `element` 的 `type` 属性生成 `DOM`，然后将 `node` 追加到 `container`中。


```js
function render(element, container) {
  const dom = document.createElement(element.type)
  
  container.appendChild(dom)
}

// const Didact = {
//   createElement,
//   render
// }

/** @jsx Didact.createElement */
// const element = (
//   <div id="foo">
//     <a>bar</a>
//     <b />
//   </div>
// );

```

---

对每个 `child` 递归就好了


```js
function render(element, container) {
  // const dom = document.createElement(element.type)
  
  ​element.props.children.forEach(child =>
      render(child, dom)
  )

  // container.appendChild(dom)
}

// const Didact = {
//   createElement,
//   render
// }

/** @jsx Didact.createElement */
// const element = (
//   <div id="foo">
//     <a>bar</a>
//     <b />
//   </div>
// );
```

---

我们需要处理下 `text` 元素，如果类型是 `TEXT_ELEMENT`，我们就创建一个 `text node`。


```js
function render(element, container) {
  const dom =
    element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type)

  //   ​element.props.children.forEach(child =>
  //       render(child, dom)
  //   )
  // container.appendChild(dom)
}
```

---

最后加上 `element` 的属性。

```js
function render(element, container) {
  // const dom =
  //   element.type == "TEXT_ELEMENT"
  //     ? document.createTextNode("")
  //     : document.createElement(element.type)

  const isProperty = key => key !== "children"
    Object.keys(element.props)
      .filter(isProperty)
      .forEach(name => {
        dom[name] = element.props[name]
      })

  // ​element.props.children.forEach(child =>
  //     render(child, dom)
  // )
  // container.appendChild(dom)
}
```

---

现在，我们有了一个自己的实现库，可以把 `JSX` 渲染到 `DOM` 上 。

在 [`codesandbox`](https://codesandbox.io/s/didact-2-k6rbj) 👈 试试

```js
function createElement(type, props, ...children) {
    return {
        type,
        props: {
            ...props,
            children: children.map(child =>
                typeof child === "object" ? child : createTextElement(child)
            )
        }
    };
}

function createTextElement(text) {
    return {
        type: "TEXT_ELEMENT",
        props: {
            nodeValue: text,
            children: []
        }
    };
}

function render(element, container) {
    const dom =
        element.type == "TEXT_ELEMENT"
            ? document.createTextNode("")
            : document.createElement(element.type);
    const isProperty = key => key !== "children";
    Object.keys(element.props)
        .filter(isProperty)
        .forEach(name => {
            dom[name] = element.props[name];
        });
    element.props.children.forEach(child => render(child, dom));
    container.appendChild(dom);
}

const Didact = {
    createElement,
    render
};

/** @jsx Didact.createElement */
const element = (
    <div style="background: salmon">
        <h1>Hello World</h1>
        <h2 style="text-align:right">from Didact</h2>
    </div>
);
const container = document.getElementById("root");
Didact.render(element, container);

```


## 第三步：`Concurrent` 模式

但...在我们添加更多的代码之前需要进行一次重构。

---

因为在 `递归` 的调用上存在一个问题。

一旦程序开始，在渲染整个 `element` 树之前是不会停下来的。如果 `element` 树过于庞大，它可能会长时间阻塞主线程。

如果浏览器需要优先处理，诸如用户输入或者保持丝滑的动画效果，就不得不等待渲染结束。

```js
function render(element, container) {
  // const dom =
  //     element.type == "TEXT_ELEMENT"
  //         ? document.createTextNode("")
  //         : document.createElement(element.type);

  // const isProperty = key => key !== "children";
  // Object.keys(element.props)
  //     .filter(isProperty)
  //     .forEach(name => {
  //         dom[name] = element.props[name];
  //     });

  element.props.children.forEach(child => 
    render(child, dom)
  );

  // container.appendChild(dom);
}
```


---

现在我们准备把工作流切分成一个个小的单元，当每一个单元结束后，如果有其他需要完成的事情，就让浏览器打断渲染。

```js
let nextUnitOfWork = null

function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    shouldYield = deadline.timeRemaining() < 1
  }
  requestIdleCallback(workLoop)
}
​
requestIdleCallback(workLoop)
​
function performUnitOfWork(nextUnitOfWork) {
  // TODO
}
```

---

用 `requestIdleCallback` 去实现循环。你可以把 `requestIdleCallback` 当成 `setTimeout`，区别在于我们不需要手动调用，当浏览器主线程空闲的时候它就会执行。

> React 现在[不再用 `requestIdleCallback`](https://github.com/facebook/react/issues/11171#issuecomment-417349573)，而是 [`schedule package`](https://github.com/facebook/react/tree/main/packages/scheduler)。但对于当前案例，原理是一样的。


```js
// let nextUnitOfWork = null

function workLoop(deadline) {
  // let shouldYield = false
  // while (nextUnitOfWork && !shouldYield) {
  //   nextUnitOfWork = performUnitOfWork(
  //     nextUnitOfWork
  //   )
  //   shouldYield = deadline.timeRemaining() < 1
  // }
  requestIdleCallback(workLoop)
}
​
requestIdleCallback(workLoop)
​
// function performUnitOfWork(nextUnitOfWork) {
  // TODO
// }
```

---

`requestIdleCallback`有一个 `deadline` 参数，可以用于检查浏览器重新掌握控制权之前我们还有多少时间。

``` js
// let nextUnitOfWork = null

function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
  //   nextUnitOfWork = performUnitOfWork(
  //     nextUnitOfWork
  //   )
    shouldYield = deadline.timeRemaining() < 1
  // }
  // requestIdleCallback(workLoop)
}
​
// requestIdleCallback(workLoop)
​
// function performUnitOfWork(nextUnitOfWork) {
  // TODO
// }
```



> 截止2019年11月，concurrent 模式不再稳定。稳定版本的循环更像是这样
``` js
while (nextUnitOfWork) {    
  nextUnitOfWork = performUnitOfWork(   
    nextUnitOfWork  
  ) 
}
```

---

在开始循环之前，我们需要设置下第一次 `unit`，然后实现一个 `performUnitOfWork` 函数。这个函数不仅要执行当前单元的 `work`，也需要返回下一个单元的 `work`。


```js
let nextUnitOfWork = null

function workLoop(deadline) {
  // let shouldYield = false
  // while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    // shouldYield = deadline.timeRemaining() < 1
  // }
  // requestIdleCallback(workLoop)
}
​
// requestIdleCallback(workLoop)
​
function performUnitOfWork(nextUnitOfWork) {
  // TODO
}
```


## 第四步：`Fibers`

为了组织这些单元的 `work`，我们需要一个数据结构：`fiber` 树 🌲。

每一个 `element` 都会对应一个`fiber`,每一个 `fiber` 也会对应一个 `work`。

用一个例子 🌰 说明下。

假设我们想渲染这样一棵 `element` 树：

```js
Didact.render(
  <div>
    <h1>
      <p />
      <a />
    </h1>
    <h2 />
  </div>,
  container
);
```

在 `render` 中，我们会创建 `root fiber`，并将它设为 `nextUnitOfWork`。剩下的工作交给 `performUnitOfWork` 函数执行。每一个 `fiber` 都要做 `3` 件事：

1. `DOM` 中添加 `element`
2. 为每一个 `element` 的 `children` 创建 `fiber`
3. 选择下一个单元的 `work`

---


这种数据结构的其中一个目的是方便找到 `work` 的下一个单元，这就是为什么每一个 `fiber` 都需要关联它的 `first child`, `next sibling` 和 `parent`

![Fiber数结构](/img/build-your-own-react-1.jpg)

---

当我们在一个 `fiber` 完成 `work`时，如果它有 `child`，将会进行下一个单元的 `work`

用我们的这个例子来说，`div fiber` 之后就到了 `h1 fiber`

---

如果当前 `fiber` 没有 `child`，就到了它的 `sibling`

举个例子，`p` 没有 `child`，所以在它完成之后就轮到 `a fiber`

---

如果既没有 `child` 也没有 `sibling`，就会去找 `uncle`，也就是 `parent sibling`。比如这个例子中的 `a` 和 `h2`。

如果 `parent` 没有 `sibling`，就继续向上查找，直到有 `sibling` 或者干脆到 `root`。至此，我们就认为在这次 `render` 中完成了所有的 `work`。

---

现在带入到代码中。

---

首先，把 `render` 函数中的代码移除。

```js
...
function render(element, container) {
  // TODO set next unit of work
}
...
```

---

我们保留部分创建 `DOM` 节点的代码，稍后会用到。


```js
function createDom(fiber) {
//   const dom =
//     fiber.type == "TEXT_ELEMENT"
//       ? document.createTextNode("")
//       : document.createElement(fiber.type)
// ​
//   const isProperty = key => key !== "children"
//   Object.keys(fiber.props)
//     .filter(isProperty)
//     .forEach(name => {
//       dom[name] = fiber.props[name]
//     })
// ​
//   return dom
}
```

---

在 `render` 函数中，我们给 `fiber` 树的 `root` 设置 `nextUnitOfWork` 

```js
function render(element, container) {
  nextUnitOfWork = {
    dom: container,
    props: {
      children: [element]
    }
  }
}

let nextUnitOfWork = null

```

---

然后，当浏览器准备好了，它会执行 `workLoop`，我们将在 `root` 开始


```js
// let nextUnitOfWork = null
​
function workLoop(deadline) {
  // let shouldYield = false
  // while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    // shouldYield = deadline.timeRemaining() < 1
  }
  // requestIdleCallback(workLoop)
}
​
// requestIdleCallback(workLoop)
​
function performUnitOfWork(fiber) {
  // TODO add dom node
  // TODO create new fibers
  // TODO return next unit of work
}
```

---

首先，我们创建一个新的 `node` 并把它追加到 `DOM` 中。

并在 `fiber.dom` 属性中追踪 `DOM` 节点。

```js
// function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
  // TODO create new fibers
  // TODO return next unit of work
}
```


---

然后对每个 `child` 创建一个新的 `fiber`

```js
// function performUnitOfWork(fiber) {
//   if (!fiber.dom) {
//     fiber.dom = createDom(fiber)
//   }
// ​
//   if (fiber.parent) {
//     fiber.parent.dom.appendChild(fiber.dom)
//   }
  const elements = fiber.props.children
  let index = 0
  let prevSibling = null
​
  while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }
  }
  // TODO return next unit of work
}
```

---

然后根据情况，我们把它作为 `child` 或者是 `subling` 来加到 `fiber` 树中。


```js
// function performUnitOfWork(fiber) {
//   if (!fiber.dom) {
//     fiber.dom = createDom(fiber)
//   }
// ​
//   if (fiber.parent) {
//     fiber.parent.dom.appendChild(fiber.dom)
//   }
//   const elements = fiber.props.children
//   let index = 0
//   let prevSibling = null
// ​
//   while (index < elements.length) {
//     const element = elements[index]
// ​
//     const newFiber = {
//       type: element.type,
//       props: element.props,
//       parent: fiber,
//       dom: null,
//     }

    if (index === 0) {
      fiber.child = newFiber
    } else {
      prevSibling.sibling = newFiber
    }
​
    prevSibling = newFiber
    index++

  }
  // TODO return next unit of work
}
```


---

最后我们去找下一个单元的 `work`。先找 `child`，然后是 `sibling`，再是 `uncle`，以此类推

```js
// function performUnitOfWork(fiber) {
//   if (!fiber.dom) {
//     fiber.dom = createDom(fiber)
//   }
// ​
//   if (fiber.parent) {
//     fiber.parent.dom.appendChild(fiber.dom)
//   }
//   const elements = fiber.props.children
//   let index = 0
//   let prevSibling = null
// ​
//   while (index < elements.length) {
//     const element = elements[index]
// ​
//     const newFiber = {
//       type: element.type,
//       props: element.props,
//       parent: fiber,
//       dom: null,
//     }

//     if (index === 0) {
//       fiber.child = newFiber
//     } else {
//       prevSibling.sibling = newFiber
//     }
// ​
//     prevSibling = newFiber
//     index++

//   }
  if (fiber.child) {
    return fiber.child
  }
  let nextFiber = fiber
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
}
```

---

这就是我们全部的 `performUnitOfWork`

```js
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
​
  const elements = fiber.props.children
  let index = 0
  let prevSibling = null
​
  while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }
​
    if (index === 0) {
      fiber.child = newFiber
    } else {
      prevSibling.sibling = newFiber
    }
​
    prevSibling = newFiber
    index++
  }
​
  if (fiber.child) {
    return fiber.child
  }
  let nextFiber = fiber
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
}
​
```


## 第五步：`Render`和 `Commit` 阶段

现在又有另一个问题。

每次作用于一个 `element` 时，我们都会给 `DOM` 添加一个新的 `node`。记住，在我们渲染整棵树之前，浏览器可以随时打断我们的 `work`。在这种情况下，用户会看到一个没有完成的 `UI`。这并不是我们想要的。

```js
function performUnitOfWork(fiber) {
//   if (!fiber.dom) {
//     fiber.dom = createDom(fiber)
//   }
​
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
​
//   const elements = fiber.props.children
//   let index = 0
//   let prevSibling = null
// ​
//   while (index < elements.length) {
//     const element = elements[index]
// ​
//     const newFiber = {
//       type: element.type,
//       props: element.props,
//       parent: fiber,
//       dom: null,
//     }
// ​
//     if (index === 0) {
//       fiber.child = newFiber
//     } else {
//       prevSibling.sibling = newFiber
//     }
// ​
//     prevSibling = newFiber
//     index++
//   }
// ​
//   if (fiber.child) {
//     return fiber.child
//   }
//   let nextFiber = fiber
//   while (nextFiber) {
//     if (nextFiber.sibling) {
//       return nextFiber.sibling
//     }
//     nextFiber = nextFiber.parent
//   }
// }
​
```

---

所以我们需要从这儿把影响 `DOM` 的这部分移除掉。

```js
function performUnitOfWork(fiber) {
//   if (!fiber.dom) {
//     fiber.dom = createDom(fiber)
//   }
​
//  >
​
//   const elements = fiber.props.children
//   let index = 0
//   let prevSibling = null
// ​
//   while (index < elements.length) {
//     const element = elements[index]
// ​
//     const newFiber = {
//       type: element.type,
//       props: element.props,
//       parent: fiber,
//       dom: null,
//     }
// ​
//     if (index === 0) {
//       fiber.child = newFiber
//     } else {
//       prevSibling.sibling = newFiber
//     }
// ​
//     prevSibling = newFiber
//     index++
//   }
// ​
//   if (fiber.child) {
//     return fiber.child
//   }
//   let nextFiber = fiber
//   while (nextFiber) {
//     if (nextFiber.sibling) {
//       return nextFiber.sibling
//     }
//     nextFiber = nextFiber.parent
//   }
// }
​
```

---

反而，我们要继续追踪这个 `fiber` 树的 `root`。我们把它叫进行中的 `work` 或者是 `wipRoot`。

> 【译者注】：`wip` 是 `work in process` 的缩写


```js
function render(element, container) {
  wipRoot = {

  }
  nextUnitOfWork = wipRoot;
  // ...
  let wipRoot = null
}
```

---

一旦我们完成了所有的 `work` (能确保的原因是不会有下一个单元的 `work`) 再将整棵 `fiber` 树提交给这个 `DOM`

---

我们把它放在 `commitRoot` 函数中。然后我们给 `dom` 递归追加所有节点。

```js
function commitRoot() {
  commitWork(wipRoot.child)
  wipRoot = null
}

function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
  domParent.appendChild(fiber.dom)
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

## 第六步：`Reconciliation`

至此，我们完成了在 `DOM` 中添加的操作，那么更新和删除呢？

这就我们准备要做的事情，

我们需要拿到 `render` 函数接收到的 `elements`，去和上一次的提交做对比。

---

所以在提交给 `DOM` 之后需要保存一份引用，把它称为 `currentRoot`。

同样我们给每个 `fiber` 添加一个 `alternate` 属性。这个属性指向 `old fiber`，就是我们上一次 `commit` 阶段提交的 `fiber`。

---

现在我们把创建 `new fiber` 的代码从 `performUnitOfWork` 中提炼出来...

...放到一个叫 `reconcileChildren` 的新函数中。

---

在这儿我们准备把 `old fiber` 和 `new element` 做一个 `reconcile`。

---

我们同时递归 `old fiber` 的子元素（`wipFiber` 的 `alternate` 属性），这个数组里面的元素就是我们想要 `reconcile` 的。

如果我们同时忽略数组中需要迭代的模版和链表，那在这个循环语句中我们剩下最多的是：`oldFiber` 和 `element`。

**`element` 是我们想要渲染给 `DOM` 的， `oldFiber` 是我们最后一次渲染的。**

我们需要比较他们，如果其中有变化，我们需要发起一次申请。

用 `type` 去比较他们：

- 如果 `old fiber` 和 `new element` 有相同的 `type`，那么我们保留 `DOM` 节点只需要更新它的 `props`

- 如果 `type` 不一样并且是一个 `new element`，意味着需要创建一个新的 `DOM` 节点

- 如果 `type` 不一样并且是一个 `old fiber`，我们需要移除这个节点

> 这里 React 用了 `keys` 属性，对于 `reconciliation` 来说会更好。例如，在一个数组中检测子元素的位置变化。

---

当 `old fiber` 和 `element` 有相同的 `type` 时，我们创建一个 `new fiber` 保存来自 `old fiber` 的 `DOM` 节点和来自 `element` 的 `props`

我们也在这个 `fiber` 中加了一个新属性：`effectTag`。这个将在 `commit` 阶段用到。

---

对于这个情况来说， `element` 需要一个新的 `DOM` 节点，我们用 `PLACEMENT` 来标记这个 `new fiber`。

---

那对于这个情况来说，我们需要删除这个 `node`，也不需要 `new fiber`，所以在 `old fiber` 标记一个 `DELETION`。

当提交 `fiber` 树的时候，我们放在正在 `work` 的 `root` 去做，因为它不存在 `old fiber`

---

所以我们需要一个数组去秩序跟踪想要移除的节点。

---

然后，当我们把变化提交给 `DOM` 时，我们也从那个数组用这些 `fiber`。

---

现在，我们修改一下 `commitWork` 函数去处理这些新的 `effectTags`。

---

如果是 `PLACEMENT`，就像往常一样，从父级的 `fiber` 追加 `DOM` 节点。

---

如果是 `DELETION`，相反，移除这个子节点。

---

如果是 `UPDATE`，我们在已存在的 `DOM` 节点上去更新变化的 `prop`。

---

我们在 `updateDOM` 这个函数中去处理。

---

对比 `new fiber` 和 `old fiber`。移除没有的 `prop`，设置新的或者有变化的 `prop`。

---

其中一个特殊的 `prop` 需要更新是 `event listener`，所以如果这个属性以 `on` 开头，就特殊处理。

---

如果 `event` 处理器有变化，移除。

---

然后添加一个新的处理器。

---

在 [`codesandbox`](https://codesandbox.io/s/didact-6-96533) 👈 尝试一下带有 `reconciliation` 的版本。


## 第七步：`Function`组件

下面需要添加的是为了支持 `Function` 组件。

我们在这个例子上做一些修改。用这个简单的 `function` 组件，去返回一个 `h1` 元素。

注意如果我们把 `JSX` 转化为 `JS`，会是这样：

``` js
function App(props) {
  return Didact.createElement(
    "h1",
    null,
    "Hi",
    props.name
  )
}

const element = Didact.createElement(App,
{
  name: "foo"
})
```

`Function` 组件有 `2` 个不同之处：

- 来自 `Function` 组件的 `fiber` 没有 `DOM` 节点。

- `children` 是从函数来的而不是直接从 `props` 获取。

---

那我们检查下 `fiber` 的 `type` 是否是一个函数，根据这个去实现不同的更新函数。

`updateHostComponent` 我们不变。

---

在 `updateFunctionComponent` 我们去获取 `children`。

在我们这个例子中，`fiber.type` 是 `App` 函数，运行之后返回 `h1` 元素。

然后，一旦我们获取到 `children`，`reconciliation` 就一样了，不需要修改任何东西。

---

`commitWork` 函数需要修改一下。

现在我们有了不带 `DOM` 节点的 `fiber`，需要有 `2` 个地方变一下。

---

第一个，为了找到一个 `DOM` 的父级，我们需要向上查找，直到一个 `fiber` 包含一个 `DOM` 节点。

---

当我们移除一个节点时，也需要找到带有 `DOM` 节点的子元素。


## 第八步：`Hooks`

到最后一步了，现在我们有了 `Function` 组件。
添加一下 `state`。

---

把我们的例子变成经典的计数组件。每次点击，它的状态都会增加。

注意现在我们用 `Didact.useState` 去获取并更新计数的值。

---

调用 `function` 组件之前，我们需要初始化一些全局变量，这样我们可以在 `useState` 函数中引用。

同时也给 `fiber` 添加了一个 `hooks` 数组，目的是支持在相同组件内多次调用 `useState`。并且我们会持续追踪当前 `hook` 的索引。

---

当我们有一个 `old hook`，并且我们没有初始化这个 `state`，那么就把 `old hook` 的 `state` 拷贝到 `new hook`。

当我们在 `fiber` 新增一个 `new hook`，那么一个一个增加 `hook` 的索引，然后返回这个 `state`。

---

`useState` 也应该返回一个可以更新 `state` 的函数，所以我们定义 `setState` 函数去接受一个行为。（在计数这个例子中，这个行为就是增加的函数）

我们把这个 `action` 追加到一个已经添加在 `hook` 的队列中。

然后我们在 `render` 函数中做一些相似的事儿，在工作的根节点中设置一个新的 `work` 作为下一个单元的 `work`，这样 `work` 循环可以开始一个新的 `render` 阶段。

---

然而我们没有运行这个 `action`。

在下次渲染组件的时候再去做，从 `old hook` 的队列中把所有的 `action` 拿出来，然后在 `new hook` 状态中一个一个去执行，所以当我们返回 `state` 的时候，它就更新了。


---

那这就是全部了，我们已经构建了自己的 `React`。

你可以在 [`codesandbox`](https://codesandbox.io/s/didact-8-21ost) 👈 或者 [`github`](https://github.com/pomber/didact) 👈 试试。



## 结语

除了帮你理解`React`是怎么实现之外，其中一个目的是想让你更深入的理解 `React` 源码。这就是为什么我们经常用一些相同的变量名和函数名。

举个例子 🌰 ，如果在真实的 `React app` 中调试你的 `function` 组件，调用栈会显示：

- `workLoop`
- `performUnitOfWork`
- `updateFunctionComponent`

我们没有包含很多 `React` 的特性和优化。例如，下面 👇 这些就和`React`中不一样。

- 在 `Didact` 中，`render` 阶段遍历了整棵树 🌲。而 `React` 会根据一些策略跳过那些没有改变的子树 🌲

- 在 `commit` 阶段，我们同样遍历了整棵树 🌲。不过 `React` 用一个链表保存了受影响的 `fiber` 并且只会访问这些 `fiber`

- 每次我们构建新的 `work` 时，我们为每个 `fiber` 都创建了一个新的 `object`。`React` 会回收这些之前树上的 `fiber`

- 在 `render` 阶段，当 `Didact` 收到更新时，它会抛弃掉正在进行的 `work` 并且从 `root` 重新开始。`React` 用过期时间标记了每个更新， 并且用它们来决定哪个更新有更高的优先级。

- 其实还有更多...

你也可以很容易的加一些特性：

- 把 `object` 用在样式属性上
- [扁平化子数组](https://github.com/pomber/didact/issues/11) 👈
- `useEffect` 的 `hook` 特性
- 通过 `key` 实现 `reconciliation`

如果你把这些或者其他特性加到 `Didact` 中，给 [GitHub 仓库](https://github.com/pomber/didact) 👈 发送一个 `PR`，这样其他人就能看见了。

感谢阅读！

