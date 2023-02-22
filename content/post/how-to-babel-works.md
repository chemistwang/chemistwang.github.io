---
# layout: post
title: "Babel工作原理"
subtitle: ""
date: 2022-08-03
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "Babel is a JavaScript compiler"
---

> Babel is a JavaScript compiler. Babal 是一个 JavaScript 编译器。


# Babel工作流

## AST

[AST在线解析](https://astexplorer.net/)

```js
// 举个例子
let a = 7;

// 转化成
{
  "type": "Program",
  "start": 0,
  "end": 193,
  "body": [
    {
      "type": "VariableDeclaration",
      "start": 181,
      "end": 191,
      "declarations": [
        {
          "type": "VariableDeclarator",
          "start": 185,
          "end": 190,
          "id": {
            "type": "Identifier",
            "start": 185,
            "end": 186,
            "name": "a"
          },
          "init": {
            "type": "Literal",
            "start": 189,
            "end": 190,
            "value": 7,
            "raw": "7"
          }
        }
      ],
      "kind": "let"
    }
  ],
  "sourceType": "module"
}
```

## 解析流程

![babel工作流](/img/how-to-babel-works-1.jpg)

### `@babel/parser` 解析

1. Lexical analysis 词法分析

> Transform the input source code into a list of **tokens**

```
var a = 7;

1. Keyword          var
2. Identifier       a          
3. Punctuator       =
4. Literal          7
5. Punctuator       ;
```

> Report errors about invalid literals or characters

```
Unterminated comment            /* var a = 7;

Unexpexted character '˚'         var a = 7˚

Expected number in radix 2      var a = 0b20;
```

2. Syntax analysis

> Transform the list of tokens into an **AST**

``` js
var a = 7;
```
``` json
{
  "type": "Program",
  ...
  "body": [
    {
      "type": "VariableDeclaration",
      ...
      "declarations": [
        {
          "type": "VariableDeclarator",
          ...
          "id": {
            "type": "Identifier",
            ...
            "name": "a"
          },
          "init": {
            "type": "Literal",
            ...
            "value": 7,
            "raw": "7"
          }
        }
      ],
      "kind": "var"
    }
  ],
  "sourceType": "module"
}

```

> Handle automatic semicolon insertion (ASI)

```
var a = foo                    var a = foo;
foo.forEach(fn)                foo.forEach(fn);

var a = foo                    var a = foo[7].forEach(n);
[7].forEach(fn)
```

> Report errors about misplaced tokens

```
Unexpected token, expexted ")"      var a = double(7;

Unexpected keyword 'if'             1 + if;
```

3. Semantic analysis

> Check the AST respects all ths static ECMAscript rules: **early errors**

``` js
Redefinition of __proto__ property

({
    __proto__: x,
    __proto__: y,
})
```
``` js
'with' in strict mode

"use strict";
with (obj) {}
```

> Report errors about invalid variables, using a **scope tracker**

Identifier 'foo' has already been declared
``` js
let foo = 2;
let foo = 3;
```

Export 'bar' is not defined
```js
{let bar = 2}
export {bar}
```

### `@babel/traverse` 转换

1. Declarative traversal

Algorithm: Depth-fist search, in-order(enter) and out-order(exit)

``` js
traverse(ast, {
    CallExpression: {
        enter() {              <--------------- enter is the default traversal order
            console.log("Function call!")
        }
    }
})
```

2. Dynamic Abstract Syntax Tree


3. Scope analysis

``` js
let MAX = 5;

export function isValid(val, big) {
    if (big) {
        const MAX = 10;
        return val < MAX;
    }
    MAX += 1;
    return val < MAX;
}
```

### `@babel/generator` 生成
