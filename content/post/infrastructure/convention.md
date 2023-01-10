---
# layout: post
title: "前端基建之（二）项目约定"
subtitle: ""
date: 2022-12-14
author: "汪洋龙"
categories: [Tech]
image: "img/coffee-bg.jpg"
description: ""
---

# 前端基建之（二）项目约定

## 1. 技术约定

- React（v18）
- React-Router V6
- React-Redux
- Typescript

## 2. 命名

- 变量禁止拼音
- 文件夹用驼峰命名

> 参考的规范链接
>
> - [凹凸实验室](https://guide.aotu.io/docs/index.html)
> - [腾讯 alloyteam 代码规范](http://alloyteam.github.io/CodeGuide/#project-naming)
> - [百度 FEX 代码规范](https://github.com/fex-team/styleguide)
> - [React 项目结构化和组件命名](https://hackernoon.com/structuring-projects-and-naming-components-in-react-1261b6e18d76)

## 3. 文件目录约定

- @api
- @assets
- @pages
- @modules
- @components
- @constant
- @hooks
- @utils
- @router
- @store
- @type
- @style

## 4. 通用工具封装

- axios
- utils
- hooks

## 5. Typescript 类型加强

[type-challenges](https://github.com/type-challenges/type-challenges)

## 6. css

- ant-desgin4.x `less`

[![Star History Chart](https://api.star-history.com/svg?repos=tailwindlabs/tailwindcss,postcss/postcss,sass/sass,stylus/stylus,less/less.js&type=Date)](https://star-history.com/#tailwindlabs/tailwindcss&postcss/postcss&sass/sass&stylus/stylus&less/less.js&Date)

## 7. 静态资源

## 8. 注释

- 注释
- README

## 9. 加密规范

## 10. 打包工具

[Tooling Report](https://bundlers.tooling.report/)

- webpack
- vite

## 11. 创建自定义 CRA Template 模板

### 11.1 新建项目

准备发布的名字就叫 `cra-template-water`

```bash
npx create-react-app cra-template-water --template typescript
```

### 11.2 构建自己的项目模板

参考资料

- [Custom Templates](https://create-react-app.dev/docs/custom-templates)
- [How to create custom Create React App (CRA) templates](https://alexgrischuk.medium.com/how-to-create-custom-create-react-app-cra-templates-73a5196edeb)

1. 按照自定义需求设定项目

2. 根目录创建 `template` 目录

```bash
mkdir template
```

3. 将根目录 `.gitignore` 复制到 `template` 目录，`template` 目录下的 `gitignore` 文件不能以 `.` 开头

```bash
cp .gitignore template/gitignore
```

4. 在根目录创建 `template.json`，放入依赖包和 `scripts`

```json
{
  "package": {
    "dependencies": {
      "@testing-library/jest-dom": "^5.16.5",
      "@testing-library/react": "^13.4.0",
      "@testing-library/user-event": "^13.5.0",
      "@types/jest": "^27.5.2",
      "@types/node": "^16.18.11",
      "@types/react": "^18.0.26",
      "@types/react-dom": "^18.0.10",
      "antd": "^5.1.2",
      "axios": "^1.2.2",
      "react": "^18.2.0",
      "react-dom": "^18.2.0",
      "react-redux": "^8.0.5",
      "react-router-dom": "^6.6.1",
      "react-scripts": "5.0.1",
      "typescript": "^4.9.4",
      "web-vitals": "^2.1.4"
    },
    "devDependencies": {
      "@commitlint/cli": "^17.4.0",
      "@commitlint/config-conventional": "^17.4.0",
      "@craco/craco": "^7.0.0",
      "eslint": "^8.31.0",
      "eslint-config-airbnb": "^19.0.4",
      "eslint-config-prettier": "^8.6.0",
      "eslint-import-resolver-typescript": "^3.5.2",
      "eslint-plugin-prettier": "^4.2.1",
      "husky": "^8.0.3",
      "prettier": "^2.8.2"
    },
    "scripts": {
      "start": "craco start",
      "build": "craco build",
      "test": "craco test",
      "eject": "craco eject",
      "lint": "eslint src --ext .ts,.jsx,.tsx --fix --quiet",
      "prettier": "prettier --write **/*.{js,jsx,ts,tsx,json}",
      "prepare": "husky install"
    }
  }
}
```

5. 将项目涉及到的文件全部拷贝到 `template` 目录下

6. 解决在 `tsconfig.json` 配置 `path` 别名后，`cra` 重新启动会覆盖 `tsconfig.json` 中的某些配置项的问题

> 问题参考
>
> - [create-react-app/issues/5645](https://github.com/facebook/create-react-app/issues/5645)
> - [create-react-app/issues/8614](https://github.com/facebook/create-react-app/issues/8614)
> - [create-react-app 启动时自动覆盖 tsconfig.js 的处理](https://juejin.cn/post/6948736395779244063)
> - [解决 React 启动自动覆盖 tsconfig 配置文件](https://blog.csdn.net/weixin_42054155/article/details/110356807)

新建 `tsconfig-path.json`

```json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@api/*": ["api/*"],
      "@assets/*": ["assets/*"],
      "@components/*": ["components/*"],
      "@hooks/*": ["hooks/*"],
      "@pages/*": ["pages/*"],
      "@router/*": ["router/*"],
      "@store/*": ["store/*"],
      "@style/*": ["style/*"],
      "@utils/*": ["utils/*"]
    }
  }
}
```

在 `tsconfig.json` 中扩展

```json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": ["src"],
  "extends": "./tsconfig-path.json"
}
```

### 11.3 通过 NPM 发布

1. 登录 npm

```bash
npm login
```

2. 如果想要类似 `@babel/parser` 的效果可以设定组织范围

```bash
npm init --scope=@carbonhydrogen2oxygen
```

3. 发布

```bash
npm publish --access public
```

4. 测试

```bash
npx create-react-app demo-admin --template @carbonhydrogen2oxygen/cra-template-water
```
