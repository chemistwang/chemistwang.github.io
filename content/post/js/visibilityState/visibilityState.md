---
layout: post
title: "快回来啊，啊好了"
subtitle: ""
date: 2022-09-23
author: "汪洋龙"
categories: []
image: "img/coffee-bg.jpg"
description: ""
---


参考资料：[changing-tab-title-focus-loss](https://fabriceleven.com/dev/changing-tab-title-focus-loss/)


顺便怒推一个 `MacOS` 的 `GIF` 制作软件：`GIPHY Capture. The GIF Maker`

浏览博客的时候，有些博主会安排下面这种效果：

![demo1](/post/js/visibilityState/1.gif)

发现很多的实现都是基于 `window` 的 `onblur` 和 `onfocus` 方法。

## onblur

``` html
<script>
    const defaultTitle = document.title;
    window.onblur = function () {
        document.title = "快回来啊";
    };
    window.onfocus = function () {
        document.title = defaultTitle;
    }
</script>
```

看似可以达到相似的效果，但是存在一个弊端，打开 `devTools` 的时候，会触发 `blur` 事件。

![demo1](/post/js/visibilityState/2.gif)

所以真正的解决方案是监听 `document` 的 `visibilitychange` 事件

## visibilitychange

```html
<script>
    const title = document.title;
    document.addEventListener("visibilitychange", function () {
        if (document.visibilityState === 'visible') {
            document.title = title
        } else {
            document.title = 'visibilitychange'
        }
    });
</script>
```

![demo1](/post/js/visibilityState/3.gif)

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/visibilitychange_event) 提供了一个很贴切的例子：

在文档可见时开始播放音乐曲目，在文档不再可见时暂停音乐。

```js
document.addEventListener("visibilitychange", function() {
  if (document.visibilityState === 'visible') {
    backgroundMusic.play();
  } else {
    backgroundMusic.pause();
  }
});
```