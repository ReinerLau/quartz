---
title: 如何发布 npm 包
tags:
  - node
---
这篇文章会带你过一遍构建和发布 npm 包的最简单流程，从始至终都会使用 `microbundle` 这个构建工具

首先先了解一点关于 `microbundle` 的知识，这个工具用在一些简单库的构建上非常高效，因为你不需要担心配置的问题，让你专注于开发你的包

下面是关于它功能的简单介绍：

- 内置配置，你只需要在 package.json 中添加 `exports` 字段
- 支持 typescript，不需要特定配置 tsconfig.json
- 多格式输出（CJS，UMD，ESM）
- 内置压缩

`microbundle` 基于 rollup 构建，如果你有更复杂的库需要构建，也许需要考虑使用纯粹的 rollup 配置, 文章下面会提及

## 初始化包

作为示例，创建一个简单的库，这个库只会导出一个 `sum` 函数，这个函数实现两个数相加的功能

创建该项目的文件夹，执行 `npm init` 命令，选择默认配置，最终生成 package.json

在 `src` 文件夹下创建 `index.ts`

```ts
export function sum(a: number, b: number) {
  return a + b;
}
```

安装 `microbundle`

```sh
pnpm add --save-dev microbundle
```

更新 package.json

```json
...
  "type": "module",
  "source": "src/index.ts", // your source code
  "exports": {
		"types": "./dist/index.d.ts", // TypeScript declaration file
    "require": "./dist/index.cjs", // CommonJS entry point
    "default": "./dist/index.esm.js" // ES Module entry point
  },
  "main": "./dist/index.cjs", // where to generate the CommonJS bundle
  "module": "./dist/index.esm.js", // where to generate the ESM bundle
  "unpkg": "./dist/index.umd.js", // where to generate the UMD bundle
  "types": "./dist/index.d.ts", // TypeScript declaration file for the package
  "scripts": {
    "build": "microbundle", // compiles "source" to "main", "module", "unpkg"
    "dev": "microbundle watch" // re-build when source files change
  }
...
```

- `source` 源码
- `exports` 暴露的模块
	- `types` 类型声明文件
	- `require` CJS 规范的入口文件
	- `default` ESM 规范的入口文件
- `main`  CJS 打包文件的存放位置
- `module` ESM 打包文件的存放位置
- `unpkg` UMD 打包文件的存放位置
- `types` 包类型声明文件位置
- `scripts` 脚本
	- `build` 将源码编译称 `main`、`module` 和 `unpkg`
	- `dev` 源码变更后重新编译

运行 `build` 脚本

```sh
pnpm build
```

最终会输出和 package.json 声明一样的文件

```
.
└── dist/
    ├── index.cjs
    ├── index.esm.js
    ├── index.umd.js
    └── index.d.ts
```

这样我们就创建了第一个包, 下面看一下更复杂的场景

## 添加 React 到你的包中

如果你想把 react 带进你的包中, 你可以继续使用 `microbundle` , 但现在你的构建命令需要变成这样:

```sh
microbundle --jsx React.createElement --jsxFragment React.Fragment --jsxImportSource react --globals react/jsx-runtime=jsx
```

添加这段命令到 package.json 中的 `build` 脚本中

```json
...
"scripts": {
  "build": "microbundle --jsx React.createElement --jsxFragment React.Fragment --jsxImportSource react --globals react/jsx-runtime=jsx"
}
...
```

## 使用 Storybook 作为 UI 组件

构建一个 UI 库时, 你也许需要一个沙盒环境来开发并预览组件, 同时在文档中提供一些示例组件

storybook 是一个带有便捷接口和构建器的沙盒, 在这里你可以很方便地描述你的组件, 每个组件状态的快照就叫做 story

在库中安装 storybook

```sh
npx storybook@latest init
```

这个命令会安装所有 storybook 需要的依赖并添加运行的脚本, 创建一个 `.storybook` 文件存放默认配置, 同时添加一些 story 示例到 `src/stories` 目录下

## 集成样式到库中

有两种定义样式的方式: 

- css 文件
- css-in-js

css 文件方便自定义样式, 但需要自成文件, css-in-js 简化了样式定义但增加了打包体积

创建一个 css 文件在 src 目录下并导入到根 react 组件文件中

```tsx
import './styles.css';

export const MySuperComponent = () => {
	return (
    <h1 className="title">Hi there!</h1>
  )
};
```

运行构建命令

```sh
pnpm build
```

导入的 `style.css` 文件就会创建到 `dist` 目录下

```
.
└── dist/
    ├── index.cjs
    ├── index.css
    ├── index.esm.js
    ├── index.umd.js
    └── index.d.ts
```

现在我们拥有了带有必要样式的 css 文件, 然而, 这个解决方案有一个不太方便的地方就是这个包被安装之后需要单独导入 css 文件

因此, 使用库的用户需要使用 css loader 来处理你的 css 文件, 像下面这样:

```tsx
import { MySuperComponent } from 'my-super-library'; 
import 'my-super-library/dist/styles.css';

export const App = () => {
  return (
    <MySuperComponent />
  );
}
```

你也可以像 `styled-components` 那样使用库, 像下面这样:

```tsx
import styled from 'styled-components';

const Title = styled.h1`
  font-size: 30px;
	font-weight: bold;
`;

export const MySuperComponent = () => {
	return (
    <Title>Hi there!</Title>
  )
};
```

这个解决方案, 用户不需要在项目中手动导入 css 文件和特定的 loader, 安装库后, 组件会自带属于自己的样式, 然而会增加打包体积且加大用户使用 css 选择器自定义元素样式的难度

以上两种方式选择适合项目的, 通常我会更喜欢 css 文件因为这允许用户自定义元素样式且不会影响打包体积, 运行速度更快

## 写一个详细的 README 文件

README.md 文件提供关于库的信息, 包括库的安装, 基础用法, 拥有的功能特性, 这通常是开发者找到你的仓库或 npm 包时第一个要看的文件, 所以这个文件需要精炼简洁

我通常喜欢按照下面顺序创建内容的结构

1. 标题
2. 简短的描述
3. 华丽的[徽章](https://shields.io/)
4. 如果是一个 UI 库, 还会包括截图或者提供一个 CodeSanbox 的示例链接
5. 功能特性列表
6. 安装指南
7. 使用方法的代码片段
8. 库的配置选项

你可以参考 [dot-path-value](https://github.com/g-makarov/dot-path-value) 和 [react-nested-dropdown](https://github.com/g-makarov/react-nested-dropdown/tree/main)

## 依赖管理

这是一个很重要的部分因为如果你做法错误, 用户可能会面临版本冲突或者其他问题, 他们可能会选择移除你的库, 所以, 来看一下不同类型依赖的主要区别:

- `devDependencies` 指定的依赖只用于开发时需要, 不会打包进打包文件的依赖, 比如如果你安装了 `microbundle` 包, 这个包用户不需要, 就把它放到 `devDependencies` 下, 它不会出现在打包文件中
-  `dependencies` 的依赖会跟随包一起安装, 包括那些会在用户项目中运行的依赖, 但比如 react, 这种用户可能已经安装的依赖, 在安装包后又重复安装了一遍会增加打包体积, 这就是为什么 `peerDependencies` 存在的原因
- `peerDenpendecies` 的依赖会在用户项目中运行, 但不会出现在包的打包文件中, 你的包会统一用户项目中存在的依赖版本, 又叫对等依赖

基本上, 我们在基于什么依赖进行开发时, 就应该指定其依赖为对等依赖, 比如你在创建一个 React 组件, 设置 react 为对等依赖, 如果开发一个带有日历的 react 组件啊, 添加 react 和日期计算库为对等依赖

如果用户的项目中没有指定的对等依赖, `npm install` 会提示警告但不会自动安装其对等依赖

## 为包使用 Github

如果你在发布一个 npm 包, 这意味着这个包可以被公共访问 (如果你是免费账户), 从用户收集反馈, 你可以为你的源码创建一个 github 仓库, 用户会在那里创建 issues 和你展开关于包的交流, 你也可以展示发布信息和获取一些小星星

你可以跳过这一步, 但在贡献开源的开发者文化中是不可或缺的一部分

## 发布并维护包

在发布你的包之前, 保证你的 package.json 文件已经适当地进行了配置这一点十分重要

第一步, 命名你的库, 尝试展示出库的核心作用

```json
"name": "react-color-picker"
```

第二步, 添加 github 仓库信息 ( 如果有的话 )

```json
...
"homepage": "https://github.com/{your_repository}",
"repository": {
    "type": "git",
    "url": "https://github.com/{your_repository}"
  },
"bugs": {
    "url": "https://github.com/{your_repository}/issues"
  },
...
```

第三步, 配置 `files` 字段

```json
...
"files": [
    "dist",
],
...
```

在你的库被安装后, 指定其在 `node_modules` 中会包含的文件, 通常来说有一个 `dist` 文件夹就足够了

第四步, 添加 `keywords` 关键字

```json
...
"keywords": ["typescript", "math", "sum", "numbers", "arithmetic", "calculator", "calculation"]
...
```

关键字是一组用于描述包的字符串, 用于在 npm 仓库中搜索和推荐, 选择那些预料中用户会用来搜索的和包相关的词

第五步, 修改 package.json 中的版本号, 默认为 `1.0.0`, 建议初版还是从 `0.0.1` 开始

```json
...
"version": "0.0.1"
...
```

第六步, 没有 npm 账号就创建一个, 在终端执行 `npm login` 命令, 按照提示授权登录 npm 账号

```sh
npm login
```

第七步, 终端执行 `npm publish` 命令就算结束了,  后续更新版本, 建议使用 `npm version patch` 来提升版本号再继续 `npm publish`

```sh
npm publish
```

```sh
npm version patch
```

## 总结

正如你所写, 创建属于自己的库并不难, 上面那些都是你创建并维护包所需要的东西, 如果你在开发时使用 mircobundle 时觉得有诸多限制, 推荐使用提供更多配置的 rollup

创建 npm 包并贡献到开源社区对提升开发者各种技能来说都是一件有价值的事情, 获取经验的同时也丰富了你的作品集
