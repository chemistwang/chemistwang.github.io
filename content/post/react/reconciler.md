---
layout: post
title: "「React源码之五」 Reconciler 工作阶段【render阶段】"
subtitle: "「源码阅读」第二层：掌握整体工作流程局部细节"
date: 2022-08-10
author: "汪洋龙"
categories: [React]
image: "img/react-bg.jpg"
tags:
  - "react"
---

## 导读

因为是 `【render 阶段】`，所以会包含 `mount（初始化）` 和 `update（更新）` 两种情况。

### 双缓存池

这个是 `React` 实现更新的一个策略。可以追溯在源码 `createWorkInProgress` 注释中。

源码位置：`react/packages/react-reconciler/src/ReactFiber.old.js` 👈

> **注释大意：** 该方法用来创建一个 alternate Fiber
>
> 我们用了一个双缓存池技术，是因为我们知道，最多需要 2 个版本的树。
> 我们共享”其他“不用的节点，这样可以自由重用。
> 这个是惰性创造的，避免给那些从未更新的对象分配额外空间。
> 如果需要，也允许我们回收额外的内存。

```js
// This is used to create an alternate fiber to do work on.
export function createWorkInProgress(current: Fiber, pendingProps: any): Fiber {
// We use a double buffering pooling technique because we know that we'll
// only ever need at most two versions of a tree. We pool the "other" unused
// node that we're free to reuse. This is lazily created to avoid allocating
// extra objects for things that are never updated. It also allow us to
// reclaim the extra memory if needed.

  let workInProgress = current.alternate;
  if (workInProgress === null) {
     current.alternate = workInProgress;
  } else {
    ...
  }
  return workInProgress;
}
```

_TODO：找找理论概念_

也就是说，在 `React` 中最多会有 `2` 棵树，`current` 树和 `workInProgress` 树。`current` 表示当前视图的树，而 `workInProgress` 表示正在内存中构建的树，两者通过 `alternate` 相互引用。当 `workInProgress` 构建完成后，会将 `current` 替换掉。

### 准备工作

结合案例是最直接的，我们用 `CRA` 的 `App` 做一个 `demo`，描述下 `Reconciler` 的工作流程。

```js
// ./src/index.js
const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// ./src/App.js
function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
      </header>
    </div>
  );
}
```

`DOM` 层级结构如下

```bash
div
  - header
    - img
    - p
      - Edit
      - code
        - src/App.js
      - and save to reload
    - a
      - Learn React
```

代码中贴上 `./src/index.js` ，是因为既然存在层级关系，自然也会包含外层 `root` 和 `React.StrictMode`，所以，实际上 `React` 的解析结构是

```bash
root
  Symbol(react.strict_mode)
    - div
      - header
        - img
        - p
          - Edit
          - code
            - src/App.js
          - and save to reload
        - a
          - Learn React
```

## 渲染流程

过程中我们重点关注下 `workInProgress` 这个变量（是一个 `FiberNode` ）以及它下面的 👇 `5` 个属性，可以在 `debug` 的时候 `watch` 一下。

- `key`
- `tag`
- `elementType`
- `type`
- `pendingProps`
- `child`
- `sibling`
- `return`
- `alternate`
- `stateNode`

### 1. performConcurrentWorkOnRoot

> 源码位置：`react/packages/react-reconciler/src/ReactFiberWorkLoop.old.js`

简写为:

```js
function performConcurrentWorkOnRoot(root, didTimeout) {
  let exitStatus = shouldTimeSlice
    ? renderRootConcurrent(root, lanes)
    : renderRootSync(root, lanes);
}
```

注意方法的 `第一个参数: root` 是通过 `bind` 绑定的

```js
performSyncWorkOnRoot.bind(null, root);
```

> 🌟 知识点：
>
> `bind` 是一个方法，创建一个新的函数。这个新函数的 `this` 被指定为 `bind()` 的 `第一个参数`，而其余参数将作为新函数的参数，供调用时使用。

我们顺便看下 `root` 的结构，这里只展示一些关键属性，它是一个 `FiberRootNode`：

```js
{
  containerInfo: div#root,
  current: FiberNode {
    tag: 3,
    alternate: ''
  },
}
```

将拿到的 `root` 作为参数传给 `renderRootSync`

> 如果想知道 `root` 如何生成，可以

### 2. renderRootSync | renderRootConcurrent

> 源码位置：`react/packages/react-reconciler/src/ReactFiberWorkLoop.old.js`

1. 调用 `prepareFreshStack` ，将 `root` 向下传
2. `prepareFreshStack` 内部调用 `createWorkInProgress` 方法，传入 `root.current` 作为参数
3. `createWorkInProgress` 方法值得拿来说

简写为：

> 该方法用来创建一个 alternate Fiber

> 我们用了一个双缓存池技术，是因为我们知道，最多需要 2 个版本的树。
> 我们共享”其他“不用的节点，这样可以自由重用。
> 这个是惰性创造的，避免给那些从未更新的对象分配额外空间。
> 如果需要，也允许我们回收额外的内存。

```js
// This is used to create an alternate fiber to do work on.
export function createWorkInProgress(current: Fiber, pendingProps: any): Fiber {
// We use a double buffering pooling technique because we know that we'll
// only ever need at most two versions of a tree. We pool the "other" unused
// node that we're free to reuse. This is lazily created to avoid allocating
// extra objects for things that are never updated. It also allow us to
// reclaim the extra memory if needed.

  let workInProgress = current.alternate;
  if (workInProgress === null) {
     current.alternate = workInProgress;
  } else {
    ...
  }
  return workInProgress;
}
```

在当前节点 `alternate` 不存在的时候，创建一个基于当前节点的 `Fiber`。

```js
current.alternate = workInProgress;
```

第一次初始化了 `workInProgress`

### 3. workLoopSync | workLoopConcurrent

> 源码位置：`react/packages/react-reconciler/src/ReactFiberWorkLoop.old.js`

区别仅在于是否需要 `shouldYield`，在当前阶段，两者没有区别

```js
function workLoopConcurrent() {
  // Perform work until Scheduler asks us to yield
  while (workInProgress !== null && !shouldYield()) {
    performUnitOfWork(workInProgress);
  }
}
function workLoopSync() {
  // Already timed out, so perform work without checking if we need to yield.
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}
```

上一步中拿到初始化的 `workInProgress`，工作流开始循环！

### 4. performUnitOfWork

> 源码位置：`react/packages/react-reconciler/src/ReactFiberWorkLoop.old.js`

可以简写为

```js
function performUnitOfWork(unitOfWork: Fiber): void {
  const current = unitOfWork.alternate;
  let next;
  next = beginWork(current, unitOfWork, renderLanes);
  if (next === null) {
    // If this doesn't spawn new work, complete the current work.
    completeUnitOfWork(unitOfWork);
  } else {
    workInProgress = next;
  }
}
```

1. 拿到 `alternate` 作为参数准备
2. 开始 `beginWork`
3. 根据 `beginWork` 的返回值是否进行 `completeUnitOfWork`

### 5. beginWork

```js
function beginWork(
    current: Fiber | null,
    workInProgress: Fiber,
    renderLanes: Lanes,
  ): Fiber | null {

    if (current !== null) {
      const oldProps = current.memoizedProps;
      const newProps = workInProgress.pendingProps;

      if (
        oldProps !== newProps ||
        hasLegacyContextChanged() ||
        ...
      ) {
        // If props or context changed, mark the fiber as having performed work.
        // This may be unset if the props are determined to be equal later (memo).
        didReceiveUpdate = true;
      } else {
        // Neither props nor legacy context changes. Check if there's a pending
        // update or context change.
        const hasScheduledUpdateOrContext = checkScheduledUpdateOrContext(
          current,
          renderLanes,
        );
        if (
          !hasScheduledUpdateOrContext &&
          // If this is the second pass of an error or suspense boundary, there
          // may not be work scheduled on `current`, so we check for this flag.
          (workInProgress.flags & DidCapture) === NoFlags
        ) {
          // No pending updates or context. Bail out now.
          didReceiveUpdate = false;
          return  (
            current,
            workInProgress,
            renderLanes,
          );
        }
        if ((current.flags & ForceUpdateForLegacySuspense) !== NoFlags) {
          // This is a special case that only exists for legacy mode.
          // See https://github.com/facebook/react/pull/19216.
          didReceiveUpdate = true;
        } else {
          // An update was scheduled on this fiber, but there are no new props
          // nor legacy context. Set this to false. If an update queue or context
          // consumer produces a changed value, it will set this to true. Otherwise,
          // the component will assume the children have not changed and bail out.
          didReceiveUpdate = false;
        }
      }
    } else {
      didReceiveUpdate = false;
    //   ...
    }

    // Before entering the begin phase, clear pending update priority.
    // TODO: This assumes that we're about to evaluate the component and process
    // the update queue. However, there's an exception: SimpleMemoComponent
    // sometimes bails out later in the begin phase. This indicates that we should
    // move this assignment out of the common path and into each branch.
    workInProgress.lanes = NoLanes;

    switch (workInProgress.tag) {
      case IndeterminateComponent:
      ...
      case LazyComponent:
      ...
      case FunctionComponent:
      ...
      case ClassComponent:
      ...
      case HostRoot:
      ...
    }
  }
```

删掉不必要的代码，就会发现 `beginWork` 先判断 `current` 是否存在：如果存在则表示当前是 `update` 阶段，根据判断条件设置 `didReceiveUpdate`，若不存在，则表示 `mount` 阶段，根据 `tag` 属性分别执行不同方法

调用 `reconcileChildren`

### 6. reconcileChildren

```js
export function reconcileChildren(
  current: Fiber | null,
  workInProgress: Fiber,
  nextChildren: any,
  renderLanes: Lanes
) {
  if (current === null) {
    // If this is a fresh new component that hasn't been rendered yet, we
    // won't update its child set by applying minimal side-effects. Instead,
    // we will add them all to the child before it gets rendered. That means
    // we can optimize this reconciliation pass by not tracking side-effects.
    workInProgress.child = mountChildFibers(
      workInProgress,
      null,
      nextChildren,
      renderLanes
    );
  } else {
    // If the current child is the same as the work in progress, it means that
    // we haven't yet started any work on these children. Therefore, we use
    // the clone algorithm to create a copy of all the current children.

    // If we had any progressed work already, that is invalid at this point so
    // let's throw it out.
    workInProgress.child = reconcileChildFibers(
      workInProgress,
      current.child,
      nextChildren,
      renderLanes
    );
  }
}
```

从整体逻辑而言，同样也是是根据 `current === null` 判断进入不同的方法。但是，细心点就会发现。它们最终进入了一个叫 `ChildReconciler` 的方法，只不过传了不同的 `boolean`。

```js
export const reconcileChildFibers = ChildReconciler(true);
export const mountChildFibers = ChildReconciler(false);
```

我们不妨看一眼 `ChildReconciler`

```js
// This wrapper function exists because I expect to clone the code in each path
// to be able to optimize each path individually by branching early. This needs
// a compiler or we can do it manually. Helpers that don't need this branching
// live outside of this function.
function ChildReconciler(shouldTrackSideEffects) {
  function reconcileChildrenArray...
  function reconcileChildrenIterator...
  function placeSingleChild...
  ...
  // This API will tag the children with the side-effect of the reconciliation
  // itself. They will be added to the side-effect list as we pass through the
  // children and the parent.
  function reconcileChildFibers...
  ...
  return reconcileChildFibers;
}
```

从参数 `shouldTrackSideEffects` 来看，表示 `是否追踪副作用`；从返回值来看，返回了其中一个叫 `reconcileChildFibers` 的函数。

在 `reconcileChildFibers` 函数中，

**总结**

在 `reconcileChildren` 阶段，`mount` 不需要 `追踪`，`update`

### 7. reconcileChildFibers

### 8. completeUnitOfWork

> 准备完成当前单元的工作，然后移动到下个 `sibling`。若没有，则返回 `parent`

```js
function completeUnitOfWork(unitOfWork: Fiber): void {
  // Attempt to complete the current unit of work, then move to the next
  // sibling. If there are no more siblings, return to the parent fiber.
}
```

### 9. completeWork

```js
function completeWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes
): Fiber | null {
  const newProps = workInProgress.pendingProps;
  // Note: This intentionally doesn't check if we're hydrating because comparing
  // to the current tree provider fiber is just as fast and less error-prone.
  // Ideally we would have a special version of the work loop only
  // for hydration.
  popTreeContext(workInProgress);
  switch (workInProgress.tag) {
    case IndeterminateComponent:
    case LazyComponent:
    case SimpleMemoComponent:
    case FunctionComponent:
  }
}
```

### 10. createElement

```js
export function createElement(
  type: string,
  props: Object,
  rootContainerElement: Element | Document | DocumentFragment,
  parentNamespace: string
): Element {
  let isCustomComponentTag;

  // We create tags in the namespace of their parent container, except HTML
  // tags get no namespace.
  const ownerDocument: Document =
    getOwnerDocumentFromRootContainer(rootContainerElement);
  let domElement: Element;
  let namespaceURI = parentNamespace;
  if (namespaceURI === HTML_NAMESPACE) {
    namespaceURI = getIntrinsicNamespace(type);
  }
  if (namespaceURI === HTML_NAMESPACE) {
    if (__DEV__) {
      isCustomComponentTag = isCustomComponent(type, props);
      // Should this check be gated by parent namespace? Not sure we want to
      // allow <SVG> or <mATH>.
      if (!isCustomComponentTag && type !== type.toLowerCase()) {
        console.error(
          "<%s /> is using incorrect casing. " +
            "Use PascalCase for React components, " +
            "or lowercase for HTML elements.",
          type
        );
      }
    }

    if (type === "script") {
      // Create the script via .innerHTML so its "parser-inserted" flag is
      // set to true and it does not execute
      const div = ownerDocument.createElement("div");
      if (__DEV__) {
        if (enableTrustedTypesIntegration && !didWarnScriptTags) {
          console.error(
            "Encountered a script tag while rendering React component. " +
              "Scripts inside React components are never executed when rendering " +
              "on the client. Consider using template tag instead " +
              "(https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template)."
          );
          didWarnScriptTags = true;
        }
      }
      div.innerHTML = "<script><" + "/script>"; // eslint-disable-line
      // This is guaranteed to yield a script element.
      const firstChild = ((div.firstChild: any): HTMLScriptElement);
      domElement = div.removeChild(firstChild);
    } else if (typeof props.is === "string") {
      // $FlowIssue `createElement` should be updated for Web Components
      domElement = ownerDocument.createElement(type, { is: props.is });
    } else {
      // Separate else branch instead of using `props.is || undefined` above because of a Firefox bug.
      // See discussion in https://github.com/facebook/react/pull/6896
      // and discussion in https://bugzilla.mozilla.org/show_bug.cgi?id=1276240
      domElement = ownerDocument.createElement(type);
      // Normally attributes are assigned in `setInitialDOMProperties`, however the `multiple` and `size`
      // attributes on `select`s needs to be added before `option`s are inserted.
      // This prevents:
      // - a bug where the `select` does not scroll to the correct option because singular
      //  `select` elements automatically pick the first item #13222
      // - a bug where the `select` set the first item as selected despite the `size` attribute #14239
      // See https://github.com/facebook/react/issues/13222
      // and https://github.com/facebook/react/issues/14239
      if (type === "select") {
        const node = ((domElement: any): HTMLSelectElement);
        if (props.multiple) {
          node.multiple = true;
        } else if (props.size) {
          // Setting a size greater than 1 causes a select to behave like `multiple=true`, where
          // it is possible that no option is selected.
          //
          // This is only necessary when a select in "single selection mode".
          node.size = props.size;
        }
      }
    }
  } else {
    domElement = ownerDocument.createElementNS(namespaceURI, type);
  }
  return domElement;
}
```

创建 `element` 元素

### 总结

至此，完整的工作流可以概括为 `2` 大阶段：`beginWork` 阶段和 `completeWork` 阶段

Step1:

| 属性           | Step1             | Step2                     | Step3                       | Step 4              |
| -------------- | ----------------- | ------------------------- | --------------------------- | ------------------- |
| `elementType`  |                   | Symbol(react.strict_mode) | f App()                     | div                 |
| `type`         |                   | Symbol(react.strict_mode) | f App()                     | div                 |
| `tag`          | 3                 | 8                         | 2                           | 5                   |
| `alternate`    |                   |                           |                             |
| `pendingProps` |                   |                           |                             |
| `所处阶段`     | beginWork         | beginWork                 | beginWork                   | beginWork           |
| `tag对应名`    | HostRoot          | Mode                      | IndeterminateComponent      | HostComponent       |
| `tag对应方法`  | updateHostRoot    | updateMode                | mountIndeterminateComponent | updateHostComponent |
|                | reconcileChildren | reconcileChildren         | reconcileChildren           | reconcileChildren   |

| 属性           | Step5             | Step6             | Step7 [img 没有 child] | Step8             |
| -------------- | ----------------- | ----------------- | ---------------------- | ----------------- |
| `elementType`  | header            | img               | -                      | p                 |
| `type`         | header            | img               | -                      | p                 |
| `tag`          | 5                 | 5                 | -                      | 5                 |
| `alternate`    |                   |                   |                        |
| `pendingProps` |                   |                   |                        |
| `所处阶段`     | beginWork         | beginWork         | completeWork           | beginWork         |
| `tag对应名`    |                   |                   |
| `tag对应方法`  |                   |                   |                        |
|                | reconcileChildren | reconcileChildren | createElement          | reconcileChildren |

| 属性           | Step9             | Step10[Edit 没有 child] | Step11                     | Step12        |
| -------------- | ----------------- | ----------------------- | -------------------------- | ------------- |
| `elementType`  | null              |                         | code                       | `code*`       |
| `type`         | null              |                         | code                       | `code*`       |
| `tag`          | 6                 |                         | 5                          | 5             |
| `alternate`    |                   |                         |                            |
| `pendingProps` | "Edit "           |                         | `{children: 'src/App.js'}` |
| `所处阶段`     | beginWork         | completeWork            | beginWork                  | completeWork  |
| `tag对应名`    |                   |                         |
| `tag对应方法`  |                   |                         |                            |
|                | reconcileChildren |                         |                            | createElement |

| 属性           | Step13                 | Step14                 | Step15        | Step16            |
| -------------- | ---------------------- | ---------------------- | ------------- | ----------------- |
| `elementType`  | null                   | null                   | p             | a                 |
| `type`         | null                   | null                   | p             | a                 |
| `tag`          | 6                      | 6                      | 5             | 5                 |
| `alternate`    |                        |                        |               |
| `pendingProps` | " and save to reload." | " and save to reload." |               |
| `所处阶段`     | beginWork              | completeWork           | completeWork  | beginWork         |
| `tag对应名`    |                        |                        |
| `tag对应方法`  |                        |                        |               |
|                | reconcileChildren      |                        | createElement | reconcileChildren |

| 属性           | Step17        | Step18        | Step19        | Step20       |
| -------------- | ------------- | ------------- | ------------- | ------------ |
| `elementType`  | a             | header        | div           | f App()      |
| `type`         | a             | header        | div           | f App()      |
| `tag`          | 5             | 5             | 5             | 0            |
| `alternate`    |               |               |               |
| `pendingProps` |               |               |               |
| `所处阶段`     | completeWork  | completeWork  | completeWork  | completeWork |
| `tag对应名`    |               |               |
| `tag对应方法`  |               |               |               |
|                | createElement | createElement | createElement |

| 属性           | Step21                    | Step22        | Step23 | Step24 |
| -------------- | ------------------------- | ------------- | ------ | ------ |
| `elementType`  | Symbol(react.strict_mode) | null          |        |
| `type`         | Symbol(react.strict_mode) | null          |        |
| `tag`          | 8                         | 3             |        |
| `alternate`    |                           |               |        |
| `pendingProps` |                           |               |        |
| `所处阶段`     | completeWork              | completeWork  |        |
| `tag对应名`    |                           |               |
| `tag对应方法`  |                           |               |        |
|                | createElement             | createElement |        |

> 注：
> `code*` 虽然存在 `src/App.js`，但并不执行它的 `beginWork | completeWork` 方法。`React`对于单一文本子节点，会特殊处理。可以移步至本文 `【扩展阅读】beginWork｜updateHostComponent` 篇。

## 扩展阅读

### root 是如何解析出来的

_TODO_

### beginWork ｜ updateHostRoot 做了什么

_TODO_

在 `beginWork` 中，

_TODO_

### beginWork ｜ updateMode 做了什么

_TODO_

在 `beginWork` 中

### beginWork ｜ updateHostComponent

在 `beginWork` 阶段，当 `workInProgress.tag = 5` 时，即本例中的

- `div`
- `header`
- `img`
- `p`
- `code`
- `a`

都会命中 🎯 本条件。

```js
function updateHostComponent(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes
) {
  pushHostContext(workInProgress);

  if (current === null) {
    tryToClaimNextHydratableInstance(workInProgress);
  }

  const type = workInProgress.type;
  const nextProps = workInProgress.pendingProps;
  const prevProps = current !== null ? current.memoizedProps : null;

  let nextChildren = nextProps.children;
  const isDirectTextChild = shouldSetTextContent(type, nextProps);

  if (isDirectTextChild) {
    // We special case a direct text child of a host node. This is a common
    // case. We won't handle it as a reified child. We will instead handle
    // this in the host environment that also has access to this prop. That
    // avoids allocating another HostText fiber and traversing it.
    nextChildren = null;
  } else if (prevProps !== null && shouldSetTextContent(type, prevProps)) {
    // If we're switching from a direct text child to a normal child, or to
    // empty, we need to schedule the text content to be reset.
    workInProgress.flags |= ContentReset;
  }

  markRef(current, workInProgress);
  reconcileChildren(current, workInProgress, nextChildren, renderLanes);
  return workInProgress.child;
}
```

TODO: 说明

其中的一段 `注释` 值得注意，大意是 `我们对节点下的文本做了特殊处理。这是一种常见的情况，我们不会把它作为具体的子元素处理，而是在能访问这个属性的宿主环境中处理。这样避免分配另一个 HostText Fiber。`

> 在本例中，这就解释了为什么 `code` 标签下还有 `src/App.js` 这个文本节点，但是却没有执行它的 `beginWork` 和 `completeWork` 方法。

### ChildReconciler

### completeWork ｜ HostComponent 做了什么

在 `completeWork` 阶段，当 `workInProgress.tag = 5` 时，即本例中的

- `div`
- `header`
- `img`
- `p`
- `code`
- `a`

都会命中 🎯 本条件。

```js
function completeWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
  const newProps = workInProgress.pendingProps;
  ...
  switch (workInProgress.tag) {
    ...
    case HostComponent: {
      ...
      const rootContainerInstance = getRootHostContainer();
      const type = workInProgress.type;
      if (current !== null && workInProgress.stateNode != null) {
        updateHostComponent(
          current,
          workInProgress,
          type,
          newProps,
          rootContainerInstance,
        );
        ...
      } else {
        if (!newProps) {
          if (workInProgress.stateNode === null) {
            ...
          }
          ...
          return null;
        }

        const currentHostContext = getHostContext();
        ...
        const wasHydrated = popHydrationState(workInProgress);
        if (wasHydrated) {
          ...
          if (
            prepareToHydrateHostInstance(
              workInProgress,
              rootContainerInstance,
              currentHostContext,
            )
          ) {
            ...
            markUpdate(workInProgress);
          }
        } else {
          const instance = createInstance(
            type,
            newProps,
            rootContainerInstance,
            currentHostContext,
            workInProgress,
          );

          appendAllChildren(instance, workInProgress, false, false);

          workInProgress.stateNode = instance;

          // Certain renderers require commit-time effects for initial mount.
          // (eg DOM renderer supports auto-focus for certain elements).
          // Make sure such renderers get scheduled for later work.
          if (
            finalizeInitialChildren(
              instance,
              type,
              newProps,
              rootContainerInstance,
              currentHostContext,
            )
          ) {
            markUpdate(workInProgress);
          }
        }
        ...
      }
      ...
      return null;
    }
    ...
}
```

`workInProgress.stateNode = instance;`

### completeWork ｜ HostText 做了什么

在 `completeWork` 阶段，当 `workInProgress.tag = 6` 时，即本例中的

- `Edit `
- `and save to reload`
- `Learn React`

都会命中 🎯 本条件。

```js
function completeWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderLanes: Lanes,
): Fiber | null {
  const newProps = workInProgress.pendingProps;
  ...
  switch (workInProgress.tag) {
    ...
    case HostText: {
      const newText = newProps;
      if (current && workInProgress.stateNode != null) {
        const oldText = current.memoizedProps;
        // If we have an alternate, that means this is an update and we need
        // to schedule a side-effect to do the updates.
        updateHostText(current, workInProgress, oldText, newText);
      } else {
        if (typeof newText !== 'string') {
          if (workInProgress.stateNode === null) {
            throw new Error(
              'We must have new props for new mounts. This error is likely ' +
                'caused by a bug in React. Please file an issue.',
            );
          }
          // This can happen when we abort work.
        }
        const rootContainerInstance = getRootHostContainer();
        const currentHostContext = getHostContext();
        const wasHydrated = popHydrationState(workInProgress);
        if (wasHydrated) {
          if (prepareToHydrateHostTextInstance(workInProgress)) {
            markUpdate(workInProgress);
          }
        } else {
          workInProgress.stateNode = createTextInstance(
            newText,
            rootContainerInstance,
            currentHostContext,
            workInProgress,
          );
        }
      }
      bubbleProperties(workInProgress);
      return null;
    }
    ...
}
```
