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

打包环境下，插件既可以使用 rollup 的原生配置和钩子，也可以使用 Vite 额外提供的配置和钩子

换句话说，单纯的 rollup 插件也可以作为 vite 插件使用

如果插件只影响打包环境，插件作为单纯的 rollup 插件可以兼容更多的非 vite 项目

如果插件需要影响开发环境，那么插件就只能作为 vite 插件在 vite 项目使用

## 基础插件

插件本质只是返回配置对象的工厂函数，写在哪里都可以

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

上面示例中，创建了一个名为 `myPlugin` 的插件，在触发 `configResolved` 钩子后获取到配置信息并打印处理，这个插件在不同环境下都会起作用

要想只作用于开发环境，配置 `apply 'serve'`

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

还可以数组形式返回配置对象，会在对应环境下触发对应配置

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

这样一个最最基本的插件就算完成了，随着插件的功能越来越复杂，可以考虑抽离成模块或者 npm 包来使用

想要实现更复杂的功能，就需要使用更多其他钩子，参考 [Vite 文档](https://vitejs.dev/guide/api-plugin?ref=hackernoon.com#universal-hooks)

## 示例

下面有一个真实场景的插件，这个插件负责抓取 `src/icons` 文件夹下的所有 `.svg` 文件，并将文件的内容整合到一个 `.svg` 文件，这个文件又称为雪碧图

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

- `configResovled` 钩子负责获取配置项，该配置项会用在接下来的钩子中
- `writeBundle` 钩子在打包流程结束后触发，创建 `sprite.svg` 文件
- `getSpriteContent` 函数负责获取 `src/icons/*.svg` 下的文件内容，这方面不是文章的重点，不需要深入了解
- 最后通过 `path.resolve()` 创建绝对路径，用于存放 `sprite.svg`

到这里，在打包环境下，这个配置是没问题的，但在开发环境下，是不会触发 `writeBundle` 钩子的，也不能托管文件，这里就需要使用服务中间件拦截对 `sprite.svg` 的请求

```ts
import { Plugin, ResolvedConfig } from 'vite';

function myPlugin(): Plugin {
  let config: ResolvedConfig;

  return {
    name: `my-plugin:serve`,
    apply: 'serve',
    async configResolved(_config) {
      config = _config;
    },
    configureServer(server) {
      return () => {
        server.middlewares.use(async (req, res, next) => { // (2)
          if (req.url !== '/sprite.svg') {
            return next();
          }
          const sprite = getSpriteContent({ pattern, prefix, svgo, currentColor });
          res.writeHead(200, {
            'Content-Type': 'image/svg+xml, charset=utf-8',
            'Cache-Control': 'no-cache',
          });
          res.end(sprite);
        });
      };
    },
  };
}
```

- `configureServer` 钩子用于配置开发服务器，它会在 Vite 内部中间件被安装之前触发，在这个示例中，我需要在内部中间件安装之后添加自定义的中间件，所以返回了一个函数
- 为了拦截对开发服务器的所有请求，使用 `server.middlewares.use()` 匹配对 `localhost:3000/sprite.svg` 的请求并模拟请求文件的行为
- 如果请求地址不是 `/sprite.svg`，就使用 `next()` 跳过，跳到下一个中间件 
- 将 `getSpriteContent` 得到的内容作为文件响应

But if files in `src/icons` are changed, deleted, or added, we should restart the server to generate new sprite content via `getSpriteContent`; for this, I will use file watching library - [chokidar](https://github.com/paulmillr/chokidar?ref=hackernoon.com). Let’s add chokidar handlers to the code