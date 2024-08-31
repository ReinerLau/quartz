---
title: 如何实现 vite 插件
tags:
  - 工程化
---
## 为什么需要实现 Vite 插件

越来越多开发者开始使用 Vite 构建项目，但毕竟较新，相较之下其相关社区和生态并没有 Webpack 那般丰富，针对自己项目中的特殊需求，也许社区中还没有相应的插件解决这些问题，因此有必要的话，可能需要开发者自己实现满足需求的插件

## Vite 的工作机制

在开始着手实现插件之前，有必要了解一下 Vite 的工作机制

首先 Vite 在不同环境下会使用不同的构建系统：

### 开发环境

执行 `vite` 命令时所处的环境

负责启动开发服务器

使用 esbuild 构建，基于 ES 模块，所以不需要将多个模块打包成一个文件也能兼容大部分现代浏览器，同时提供更快的热更新

### 打包环境

执行 `vite build` 命令时所处的环境，

负责输出打包文件

使用 rollup 构建，灵活且具有丰富的生态，可以输出不同规范且高度优化过的打包文件

### 插件接口

开发环境下，插件既可以使用 rollup 的原生配置和钩子，也可以使用 Vite 额外提供的配置和钩子

换句话说，单纯的 rollup 插件也可以作为 vite 插件使用

如果插件只影响打包环境，插件作为单纯的 rollup 插件可以兼容更多的非 vite 项目

如果插件需要影响开发环境，那么插件就只能作为 vite 插件在 vite 项目使用

## 创建基础插件

插件本质只是单纯的返回对象的工厂函数，写在哪里都可以

```ts
// vite.config.ts
import { defineConfig, Plugin } from 'vite';

function myPlugin(): Plugin {
  return {
    name: 'my-plugin',
    configResolved(config) {
      console.log(config);
	},
  };
}

export default defineConfig({
  plugins: [
    myPlugin(),
  ],
});
```

上面示例：
- 插件名为 `myPlugin`
- 打印 Vite 的配置
	- 在控制台中
	- 解析配置后马上打印
	- 在不同环境下都起作用

只作用于开发环境，配置 `apply 'serve'`

```ts
// vite.config.ts
import { defineConfig, Plugin } from 'vite';

function myPlugin(): Plugin {
  return {
    name: 'my-plugin',
	apply: 'serve',
    configResolved(config) {
      console.log(config);
	},
  };
}

export default defineConfig({
  plugins: [
    myPlugin(),
  ],
});
```

只作用于打包环境，配置 `apply: 'build'`

支持返回数组，适用于在不同环境下实现不同的功能

```ts
// vite.config.ts
import { defineConfig, Plugin } from 'vite';

function myPlugin(): Plugin[] {
  return [
    {
      name: 'my-plugin:serve',
      apply: 'serve',
      configResolved(config) {
        console.log('dev server:', config);
      },
    },
    {
      name: 'my-plugin:build',
      apply: 'build',
      configResolved(config) {
        console.log('bundle:', config);
      },
    },
  ];
}

export default defineConfig({
  plugins: [
    myPlugin(),
  ],
});
```

> 如果数组中多个配置使用相同的 `apply`，只会应用最后一个


插件变复杂：
- 抽离成模块
- 抽离成包

钩子：
- 更多钩子看 Vite 文档
- 先拆解我的示例

## 拆解插件

基于 icon 文件创建  SVG 雪碧图的[插件](https://github.com/g-makarov/vite-plugin-svg-spritemap?ref=hackernoon.com)：
- 抓取 `src/icons` 文件夹下所有 svg 图标文件
- 收集内容到单个 svg 文件中，即 svg 雪碧图

```ts
import { Plugin, ResolvedConfig } from 'vite';
import path from 'path';
import fs from 'fs-extra';

function myPlugin(): Plugin {
  let config: ResolvedConfig;

  return {
    name: 'my-plugin:build',
    apply: 'build',
    async configResolved(_config) {
      config = _config;
    },
    writeBundle() {
      const sprite = getSpriteContent({ pattern: 'src/icons/*.svg' });
      const filePath = path.resolve(config.root, config.build.outDir, 'sprite.svg');
      fs.ensureFileSync(filePath);
      fs.writeFileSync(filePath, sprite);
    },
  };
}
```

`configResovled` 钩子获取配置项并用在下一个钩子中

`writeBundle` 钩子在打包流程结束后触发，在这里创建 `sprite.svg` 文件

`getSpriteContent` 函数返回基于 `src/icons/*.svg` 下的雪碧图字符串，这里不深入讲解

然后，我通过 `path.resolve()` 创建存放 `sprite.svg` 的绝对路径来