---
# layout: post
title: "sourcemap中的Base64VLQ"
subtitle: ""
date: 2022-08-11
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "Base64VLQ的编码与解码"
---

## sourcemap

参考资料：

- [Introduction to JavaScript Source Maps](https://developer.chrome.com/blog/sourcemaps/)

- [Source Map Revision 3 Proposal](https://docs.google.com/document/d/1U1RGAehQwRypUTovF1KRlpiOFze0b-_2gc6fAH0KY0k/edit?hl=en_US&pli=1&pli=1)

### 文件结构

``` js
{
    "version": 3,
    "file": 'react.development.js',
    "sources": [
        "/Users/wyl/github/debug-react/react/packages/shared/ReactVersion.js",
        "/Users/wyl/github/debug-react/react/packages/shared/ReactSymbols.js",
        "/Users/wyl/github/debug-react/react/packages/react/src/ReactCurrentDispatcher.js",
        ...
    ],
    "sourcesContent": [
        "/**\n * Copyright (c) Facebook, Inc. and its affiliates.\n *\n * This source code is licensed under the MIT license found in the\n * LICENSE file in the root directory of this source tree.\n */\n\n// TODO: this is special because it gets imported during build.\n//\n// TODO: 18.0.0 has not been released to NPM;\n// It exists as a placeholder so that DevTools can support work tag changes between releases.\n// When we next publish a release, update the matching TODO in backend/renderer.js\n// TODO: This module is used both by the release scripts and to expose a version\n// at runtime. We should instead inject the version number as part of the build\n// process, and use the ReactVersions.js module as the single source of truth.\nexport default '18.2.0';\n",
        ...
    ],
    "names": [
        "REACT_ELEMENT_TYPE",
        "Symbol",
        "for",
        "REACT_PORTAL_TYPE",
        "REACT_FRAGMENT_TYPE",
        "REACT_STRICT_MODE_TYPE",
        ...
    ],
    "mappings": ";;;;;;EAOA;EACA;EACA;EACA;EACA;EACA;EACA;EACA;AACA,qBAAe,QAAf;;ECNA;..."
}
```

- `version`: 版本
- `file`: 转换后的文件名
- `sourceRoot`: 源文件目录前缀
- `sources`: 转换前的文件列表
- `sourcesContent`: ''
- `names`: 转换前的所有变量名和属性名
- `mappings`: 记录位置信息的字符串

### 有趣的mappings属性

最初看到这一串字符串是一脸懵逼，甚至觉得连续的分号：`";;;;;;EAOA..."` 什么鬼，非常不正常。

接下来不妨看看[官方的描述](https://developer.chrome.com/blog/sourcemaps/#base64-vlq-and-keeping-the-source-map-small) 👈，解释了为什么这么用。（可以在这篇翻译的文章后面查看原文的思路）


> **Base64 VLQ 维持了 sourcemap 的轻量化【意译】**
>
> 最初，`source map` 对所有映射关系有了非常详细的输出，最终的结果就是它生成的代码大小达到 `10` 倍之多。在版本 `2`中缩减了 `50%`， 然后版本 `3` 再次缩减了 `50%`，对于一个 `133kb` 的文件最终会得到一个大约 `300kb` 的 `source map`。那它们是如何缩减了大小还能维持复杂的映射关系呢？
>
> `VLQ（可变长度数量）` 会和 `Base64` 编码一起使用。 `mappings` 属性是一个超级大的字符串。在这个字符串中，分号 `;` 代表生成文件的行号。每一行都有逗号 `,` 表示那一行中的每个 `segment`。每个 `segment` 都有 `1,4,5` 不等的长度。有一些看起来会更长一点，这些包含了连续的比特位。每个 `segment` 都建立在前一个 `segment` 的基础上，这有助于减小文件大小。 
>
> ![AAgBC](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/s9OBEzuUo3o5HLyKsnc7.png?auto=format&w=650)
>
> 【注】[Base编码对照表](https://en.wikipedia.org/wiki/Base64) 👈
>
> 像我上面提到的每个片段可能会有 `1,4,5` 不等的长度。这个图表考虑用一个 `4` 位的可变长度加上一个连续比特位（`g`）。
> 我们把这个 `segment` 拆解开，看来下 `source map` 是如何解决原始位置的问题。字符上面的值（`A => 0,A => 0,g => 32,B => 1,C => 2`）纯粹是用 `Base64` 解码的值，需要有一些步骤去得到他们真正的值。每个 `segment` 通常解决了下面 `5` 件事儿：
>
> - 生成的列
> - 出现的原始文件
> - 原始行号
> - 原始列
> - 和 如果存在原始名称
>
> 并不是每个 `segment` 都有名字，方法名或者参数，所以 `segment` 通常在 `4` 到 `5` 个长度之间切换。 上面图表那个 `segment` 中的 `g` 是所谓的连续比特位，这允许在 `Base64 VLQ` 解码阶段进一步优化。连续比特位允许你构建一个 `segment` 的值去存储一个很大的数字而不用特意有一个空间，这是一种非常聪明节省空间的技术，可以追溯到 `midi` 格式。
>
> 上图的 `AAgBC` 进一步处理之后将返回 `0,0,32,16,1` —— `32` 作为连续比特位可以帮助构建下一个 `16` 那个值。`B` 用 `Base64` 解码之后是 `1`。所以能用到重要的值是 `0,0,16,1`。这就让我们知道在生成的 `map` 文件中第 `1` 行（行是通过分号计数的）第 `0` 列对应的是第 `0` 个文件（数组中的第 `0` 个文件是 `foo.js`），第 `16` 行第 `1` 列。
> 
> 为了展示下 `segment` 是如何解码的，我会引用下 `Mozilla` 的 [`Source Map JavaScript library`](https://github.com/mozilla/source-map/) 👈。你也可以看下 `WebKit` 的开发者工具 [`source mapping code`](https://code.google.com/codesearch#OAMlx_jo-ck/src/third_party/WebKit/Source/WebCore/inspector/front-end/CompilerSourceMapping.js) 👈，它也是用 `JavaScript` 写的。
>
> 为了正确理解我们是如何从 `B` 得到的 `16`，我们需要对位运算符有一个基础的理解，然后看下是如何作用于源码的映射。通过按位与（`&`）比较数字（`32`）和 `VLQ_CONTINUATION_BIT` (二进制100000或者32)， 前面那个 `g` 被标记作为连续比特位。
> 
> 
> ```
> 32 & 32 = 32
> // or
> 100000
> |
> |
> V
> 100000 
> ```
> 
> 每一位都出现 `1` 才会返回 `1`。所以如上图所示， `33 & 32` 解码的值也是 `32`，因为它们仅共享 `32` 位位置。然后对于每个前面的连续位，都会左移增加 `5` 位。在上述情况下，它只移动 `5` 次，因此将 `1` （`B`）左移 `5`。
> 
> ```
> 1 << 5 // 32
> 
> // Shift the bit by 5 spots
> ______
> |    |
> V    V
> 100001 = 100000 = 32
> ```
>
> 然后将数字（`32`）右移一位，这个值就从一个 `VLQ` 变过来了。
>
> ```
> 32 >> 1 // 16
> //or
> 100000
> |
>  |
>  V
> 010000 = 16
> ```
> 
> 所以我们得出：是如何从 `1` 变成 `16` 的。看起来是个复杂的过程，不过一旦这个数字开始变大，就会变得很有意义。


**上文大体思路 👆**：

```
{
    version : 3,
    file: "out.js",
    sourceRoot : "",
    sources: ["foo.js", "bar.js"],
    names: ["src", "maps", "are", "fun"],
    mappings: "AAgBC,SAAQ,CAAEA" ⬅️ 因为规定按分号（；）切分，所以表示打包文件 out.js 的第 1 行
}
```

1. `Base64 + VLQ` 是有 `编码` 和 `解码` 两个过程，文章描述的是 `解码` 过程
1. 从上面的 `map` 文件中举例说明了 `AAgBC`，如果用纯粹的 `Base64` 解码得到的值是 `0 0 32 1 2`
2. 但这并不是真正的值，需要一些步骤才能得到
3. 真正的值是 `0 0 16 1` 。（表示当前行第 `0` 列对应第 `0`个原文件的第 `16` 行 第 `1` 列）
4. 



[在线BASE64 VLQ网站](https://www.murzwin.com/base64vlq.html) 👈

### 实现

在 `JavaScript` 中，有两个函数被用来处理解码和编码 `base64` 字符串：

- atob() 解码
- btoa() 编码

```js
btoa('a')       // YQ==
atob('YQ==')    // a
```