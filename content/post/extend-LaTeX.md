---
# layout: post
title: "Hugo 主题添加 Latex 支持"
subtitle: ""
date: 2023-03-27
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: ""
mathjax: true
---

# Hugo 主题添加 Latex 支持

最近记录机器学习相关笔记时，需要用到很多数学公式，在现有的 `Hugo` 主题框架上扩展一下 `Latex` 的语法支持。

## 1. 添加脚本

在已有的主题文件夹下 `themes/hugo-theme-cleanwhite/layouts/partials` 创建 `mathjax_support.html`，添加如下代码。

```html
<script>
  MathJax = {
    tex: {
      inlineMath: [
        ["$", "$"],
        ["\\(", "\\)"],
      ],
      displayMath: [
        ["$$", "$$"],
        ["\\[", "\\]"],
      ],
      processEscapes: true,
      processEnvironments: true,
    },
    options: {
      skipHtmlTags: ["script", "noscript", "style", "textarea", "pre"],
    },
  };

  window.addEventListener("load", (event) => {
    document.querySelectorAll("mjx-container").forEach(function (x) {
      x.parentElement.classList += "has-jax";
    });
  });
</script>
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script
  type="text/javascript"
  id="MathJax-script"
  async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"
></script>
```

## 2. 添加 Hugo 渲染

在 `themes/hugo-theme-cleanwhite/layouts/partials/head.html` 中添加解析语法，注意解析语法要放在 `head` 标签之内。

```html
<head>
  ...

  <!-- LaTeX  MathJax 3 -->
  {{ if .Params.mathjax }}{{ partial "mathjax_support.html" . }}{{ end }}
</head>
<style>
  code.has-jax {
    -webkit-font-smoothing: antialiased;
    background: inherit !important;
    border: none !important;
    font-size: 100%;
  }
</style>
```

## 3. 在所在的 md 文件中添加 `mathjax: true`

```md
---
# layout: post
title: "《PyTorch 深度学习实践》学习笔记"
subtitle: ""
date: 2023-03-24
author: "汪洋龙"
categories: [Tech]
image: "img/ai.jpg"
description: ""
mathjax: true // 👈 添加
---
```

## 4. 使用

行内公式语法用单 `$` 括起来:

```bash
$\hat{y} =x\  \times w\  +\  b$
```

`$\hat{y} =x\  \times w\  +\  b$`

行间公式语法用双 `$` 括起来:

```bash
$$\hat{y} =x\  \times w\  +\  b$$
```

`$$\hat{y} =x\  \times w\  +\  b$$`

> MacOS 用户推荐 `App Store` 的 `XFormula` 编写并输出 `Latex`

## 5. 可以发布成自己的 Hugo 主题

[Github hugo-theme-cleanwhite-extend](https://github.com/chemistwang/hugo-theme-cleanwhite-extend)

```bash
git submodule add https://github.com/chemistwang/hugo-theme-cleanwhite-extend themes/hugo-theme-cleanwhite-extend
```
