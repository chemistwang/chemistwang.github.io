---
layout: post
title: "navigator.clipboard"
subtitle: ""
date: 2022-12-05
author: "汪洋龙"
categories: []
image: "img/coffee-bg.jpg"
description: "本地可以，线上却不行"
---

本地实现一个粘贴板功能的时候，用了 `navigator.clipboard` 却在线上环境无法使用。

原因： 因为 `navigator.clipboard` 只能在安全网络环境中才能使用，也就是，安全域包括本地访问与开启 TLS 安全认证的地址，如 `https` 协议的地址、`127.0.0.1` 或 `localhost` 才能正常使用。

非安全环境下的协议地址，例如 `http`

```js
async function copyTextToClipboard(text: string) {
  if ("clipboard" in navigator) {
    const res = await navigator.clipboard.writeText(text);
    return res;
  }
  const textarea = document.createElement("textarea");
  document.body.appendChild(textarea);
  // 隐藏此输入框
  textarea.style.position = "fixed";
  textarea.style.clip = "rect(0 0 0 0)";
  textarea.style.top = "10px";
  // 赋值
  textarea.value = text;
  // 选中
  textarea.select();
  // 复制
  document.execCommand("copy", true);
  // 移除输入框
  document.body.removeChild(textarea);
}
```
