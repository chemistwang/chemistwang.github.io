---
# layout: post
title: "前端基建之（一）代码风格和提交规范配置"
subtitle: ""
date: 2022-12-13
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: "Eslint Prettier Commitlint Husky"
---

# 前端基建之（一）代码风格和提交规范配置

## 1. 初始化项目

```bash
npx create-react-app demo-framework --template typescript
```

## 2. 配置 Eslint

### 2.1 `vscode` 安装 `eslint`

### 2.2 安装依赖

```bash
npm i eslint -D
```

### 2.3 初始化 eslint

```bash
npx eslint --init

You can also run this command directly using 'npm init @eslint/config'.
npx: installed 41 in 3.304s
✔ How would you like to use ESLint? · problems
✔ What type of modules does your project use? · esm
✔ Which framework does your project use? · react
✔ Does your project use TypeScript? · No / Yes ✔
✔ Where does your code run? · browser
✔ What format do you want your config file to be in? · JavaScript
The config that you've selected requires the following dependencies:

eslint-plugin-react@latest @typescript-eslint/eslint-plugin@latest @typescript-eslint/parser@latest
✔ Would you like to install them now? · No ✔ / Yes
A config file was generated, but the config file itself may not follow your linting rules.
```

```js
module.exports = {
  env: {
    browser: true,
    es2021: true,
  },
  extends: [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:@typescript-eslint/recommended",
  ],
  overrides: [],
  parser: "@typescript-eslint/parser",
  parserOptions: {
    ecmaVersion: "latest",
    sourceType: "module",
  },
  plugins: ["react", "@typescript-eslint"],
  rules: {},
};
```

### 2.4 社区规范

三种社区主流规范

- Airbnb
- Standard
- Google

[![Star History Chart](https://api.star-history.com/svg?repos=google/eslint-config-google,standard/standard,airbnb/javascript&type=Date)](https://star-history.com/#google/eslint-config-google&standard/standard&airbnb/javascript&Date)

### 2.5 配置 airbnb 规范

1. 安装 `eslint-config-airbnb`

```bash
npm i eslint-config-airbnb -D
```

2. 在 `.eslintrc.js` 配置规则

```json
//  .eslintrc.js
{
    ...
    "extends": [
        "airbnb",
        ...
    ]
    ...
}
```

3. `package.json` 配置 `script`

```json
...
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "lint": "eslint src --ext .ts,.jsx,.tsx --fix --quiet"
  },
...
```

4. 修复运行错误

- `error  Unable to resolve path to module './App'        import/no-unresolved`

```bash
npm i eslint-import-resolver-typescript -D
```

```js
// .eslintrc
...
  rules: {},
  settings: {
    "import/resolver": {
      typescript: {
        directory: "./tsconfig.json",
      },
    },
  },
...
```

- `error  Missing file extension for "./App"              import/extensions`

```js
// .eslintrc
...
  rules: {
    "react/jsx-filename-extension": [
      2,
      {
        extensions: [".js", ".jsx", ".ts", ".tsx"],
      },
    ],
  },
...
```

- `error  JSX not allowed in files with extension '.tsx'  react/jsx-filename-extension`

```js
// .eslintrc
  rules: {
    "import/extensions": [
      "error",
      "ignorePackages",
      {
        js: "never",
        jsx: "never",
        ts: "never",
        tsx: "never",
      },
    ],
  },
```

- `error  Do not use a triple slash reference for react, use `import` style instead  @typescript-eslint/triple-slash-reference`

```js
// .eslintrc
  rules: {
    "@typescript-eslint/triple-slash-reference": "warn",
  }
```

## 3. prettier

[Prettier 官网](https://prettier.io/)

### 3.1 跟 Linters 区别

[Prettier vs. Linters](https://www.prettier.cn/docs/comparison.html)

**In other words, use Prettier for formatting and linters for catching bugs!**

### 3.2 `vscode` 安装 `prettier`

### 3.3 安装配置

`VsCode` 安装 `Prettier - Code formatter`

### 3.4. 规则设定

| 标签名                                      | 含义             | 属性                         | 默认值      |
| ------------------------------------------- | ---------------- | ---------------------------- | ----------- |
| Print Width                                 | 行宽度           | `printWidth`                 | `80`        |
| Tab Width                                   | Tab 宽度         | `tabWidth`                   | `2`         |
| Tabs                                        | 是否使用 Tab     | `useTabs`                    | `false`     |
| Semicolons                                  | 是否句末开启分号 | `semi`                       | `true`      |
| Quotes                                      | 是否启用单引号   | `singleQuote`                | `false`     |
| Quote Props                                 |                  | `quoteProps`                 | `as-needed` |
| JSX Quotes                                  | JSX 是否用单引号 | `jsxSingleQuote`             | `false`     |
| Trailing Commas                             | 尾部逗号         | `trailingComma`              | `es5`       |
| Bracket Spacing                             | 括号空格         | `bracketSpacing`             | `true`      |
| Bracket Line                                | 括号换行         | `bracketSameLine`            | `false`     |
| Arrow Function Parentheses                  | 剪头函数括号     | `arrowParens`                | `always`    |
| Range                                       | 格式化范围       | `rangeStart` `rangeEnd`      |
| Parser                                      | 指定解析器       | `parser`                     |
| File Path                                   |                  | `filepath`                   | ``          |
| Require Pragma                              |                  | `requirePragma`              | `false`     |
| Insert Pragma                               |                  | `insertPragma`               | `false`     |
| Prose Wrap                                  |                  | `proseWrap`                  | `preserve`  |
| HTML Whitespace Sensitivity                 |                  | `htmlWhitespaceSensitivity`  | `css`       |
| Vue files script and style tags indentation |                  | `vueIndentScriptAndStyle`    | `false`     |
| End of Line                                 |                  | `endOfLine`                  | `lf`        |
| Embedded Language Formatting                |                  | `embeddedLanguageFormatting` | `auto`      |
| Single Attribute Per Line                   |                  | `singleAttributePerLine`     | `false`     |

### 3.5 案例

- [React](https://github.com/facebook/react/blob/main/.prettierrc.js)

```js
"use strict";

const { esNextPaths } = require("./scripts/shared/pathsByLanguageVersion");

module.exports = {
  bracketSpacing: false,
  singleQuote: true,
  jsxBracketSameLine: true,
  trailingComma: "es5",
  printWidth: 80,
  parser: "babel",

  overrides: [
    {
      files: esNextPaths,
      options: {
        trailingComma: "all",
      },
    },
  ],
};
```

- [Ant Design](https://github.com/ant-design/ant-design/blob/master/.prettierrc)

```json
{
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "proseWrap": "never",
  "overrides": [
    {
      "files": ".prettierrc",
      "options": {
        "parser": "json"
      }
    }
  ]
}
```

- [Angular](https://github.com/angular/angular/blob/main/.prettierrc)

```json
{
  "printWidth": 100,
  "tabWidth": 2,
  "tabs": false,
  "singleQuote": true,
  "semicolon": true,
  "quoteProps": "preserve",
  "bracketSpacing": false
}
```

- [Vue](https://github.com/vuejs/vue/blob/main/.prettierrc)

```json
semi: false
singleQuote: true
printWidth: 80
trailingComma: 'none'
arrowParens: 'avoid'
```

### 3.6 配置 prettier

1.  安装

```bash
npm i prettier -D
```

2. 根目录创建 `.prettierrc`

```json
{
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "singleQuote": true
}
```

3. `package.json` 新增命令

```json
...
"scripts": {
    ...
    "prettier": "prettier --write **/*.{js,jsx,ts,tsx,json}"
    ...
}
...
```

### 3.7 解决和 `eslint` 的冲突

因为 `eslint` 也可以做风格检查，所以会跟 `prettier` 冲突

> 例如：在 `prettier` 中设置 `"semi": false`，在 `eslintrc` 中设置 `semi: 2`。保存文件会取消分号，但是 `npm run lint` 之后又会新增分号。

1. 将 `prettier` 集成到 `eslint` 中

```bash
npm i eslint-config-prettier -D # 关闭可能与prettier冲突的规则
npm i eslint-plugin-prettier -D # 使用prettier代替eslint格式化
```

2. 设置 `prettier` 为默认格式化并设置保存时生效

在 `.eslintrc` 中配置 `prettier`

```js
...
  extends: [
    'airbnb',
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:@typescript-eslint/recommended',
    'prettier'
  ],
  ...
```

## 4. 配置别名

### 4.1 `@craco/craco`

安装

```bash
npm i -D @craco/craco
```

根目录新建 `craco.config.js`

```js
const path = require("path");

const pathResolve = (pathUrl) => path.join(__dirname, pathUrl);

module.exports = {
  webpack: {
    alias: {
      "@": pathResolve("src"),
      "@assets": pathResolve("src/assets"),
      "@components": pathResolve("src/components"),
      "@utils": pathResolve("src/utils"),
    },
  },
};
```

修改 `tsconfig.json` 配置文件

```json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@utils/*": ["utils/*"]
    }
  }
  //   ...
}
```

修改 `package.json`

```json
...
  "scripts": {
    "start": "craco start",
    "build": "craco build",
    "test": "craco test",
    "eject": "craco eject"
  },
...
```

### 4.2 `webpack`

暴露 `webpack` 配置

```bash
npm run eject
```

修改 `config/webpack/webpack.config.js`

```js
...
    alias: {
    // Support React Native Web
    // https://www.smashingmagazine.com/2016/08/a-glimpse-into-the-future-with-react-native-for-web/
    "react-native": "react-native-web",
    // Allows for better profiling with ReactDevTools
    ...(isEnvProductionProfile && {
        "react-dom$": "react-dom/profiling",
        "scheduler/tracing": "scheduler/tracing-profiling",
    }),
    ...(modules.webpackAliases || {}),
    "@utils": path.resolve(__dirname, "../src/utils"),
    },
...
```

修改 `tsconfig.json` 配置文件

```json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@utils/*": ["utils/*"]
    }
  }
  //   ...
}
```

## 5. Husky

用 `git hooks` 自动化校验

### 5.1 安装 husky

```bash
npm i husky -D
```

### 5.2 初始化 husky

```bash
npx husky install
```

### 5.3 添加 lint 钩子

```bash
npx husky add .husky/pre-commit "npm run lint"
```

## 6. Commitlint

约定 `commit` 信息

### 6.1 安装

- [@commitlint/cli](https://commitlint.js.org/#/) `commitlint` 命令行工具
- [@commitlint/config-conventional](https://www.npmjs.com/package/@commitlint/config-conventional) 基于 `Angular` 的约定规范

```bash
npm i @commitlint/cli @commitlint/config-conventional -D
```

### 6.2. 创建 `.commitlintrc`，写入配置

```json
{
  "extends": ["@commitlint/config-conventional"]
}
```

### 6.3. 集成 `husky` 中

```bash
npx husky add .husky/commit-msg 'npx --no-install commitlint --edit "$1"'
```

- `feat`: 新功能
- `fix`: 修补 BUG
- `docs`: 修改文档，eg：README，CHANGELOG，CONTRIBUTE 等
- `style`: 不改变代码逻辑（仅仅修改空格、格式缩进、逗号等）
- `refactor`: 重构（既不修复错误也不添加功能）
- `perf`：优化相关，比如提升性能、体验
- `test`：增加测试，包括单元测试、集成测试等
- `build`：构建系统或外部依赖项的更改
- `ci`：自动化流程配置或脚本修改
- `chore`：非 src 和 test 的修改，发布版本等
- `revert`：恢复先前的提交

## 7. Semantic Release
