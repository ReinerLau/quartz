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

现在如果 `src/icons` 中的文件发生修改，删除或者新增，我们需要重启开发服务器才能重新执行 `getSpriteContent` 生成雪碧图，为了解决这个问题，使用 [chokidar](https://github.com/paulmillr/chokidar?ref=hackernoon.com)

```ts
import { Plugin, ResolvedConfig } from 'vite';
import chokidar from 'chokidar';

function myPlugin(): Plugin {
  let config: ResolvedConfig;
  let watcher: chokidar.FSWatcher; 

  return {
    name: `my-plugin:serve`,
    apply: 'serve',
    async configResolved(_config) {
      config = _config;
    },
    configureServer(server) {
	  function reloadPage() {
        server.ws.send({ type: 'full-reload', path: '*' });
      }

      watcher = chokidar
        .watch('src/icons/*.svg', {
          cwd: config.root,
          ignoreInitial: true,
        })
        .on('add', reloadPage)
        .on('change', reloadPage)
        .on('unlink', reloadPage); 

      return () => {
        server.middlewares.use(async (req, res, next) => {
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

正如你所见，插件的创建并不复杂，你只需要从 Vite 或者 Rollup 中找到符合需求的钩子，比如在这个示例中，使用了来自 rollup 的 `writeBundle` 和来自 vite 的 `configureServe`，因为 rollup 不支持开发环境

示例中的 `writeBundel` 非常简单，拿到雪碧图内容并放进文件中，按理说在开发环境下也能实现相同的功能，我看了一下其他开发者写的插件，他们也是用服务器中间件处理的，使用 `configureServer` 的 `server` 参数，添加中间件拦截所有对开发服务器 `sprite.svg` 的请求

## 钩子

前面提到过，想要创建更复杂的插件，你需要浏览一下其他钩子，这些钩子都在文档中都有描述，分为[通用钩子](https://vitejs.dev/guide/api-plugin?ref=hackernoon.com#universal-hooks)和[vite 特定钩子](https://vitejs.dev/guide/api-plugin?ref=hackernoon.com#vite-specific-hooks)

## 命名

在插件的命名方面，vite 提供了一些规范：

- 需要带有 `vite-plugin-` 的前缀
- package.json 中包括 `vite-plugin` 关键字
- 在插件介绍中解释为什么这是只适用于 vite 的插件
- 如果插件只适用于特定框架，包括前缀中添加，比如 `vite-plugin-vue-`, `vite-plugin-react-`, `vite-plugin-svelte-`

## 发布

推荐你发布插件到 npm 仓库上，因为这是分享知识是 IT 社区的基本原则，促进互相成长，发布的方式参考[[Node.js/npm/如何发布 npm 包|这里]]

同时推荐你提交插件到 vite 的[社区](https://github.com/vitejs/awesome-vite#plugins)，各地开发者会在这里寻找适合自己的插件，这是一个很好贡献 vite 生态的机会，提交的流程也很简单，参考[这里](https://github.com/vitejs/awesome-vite/blob/master/.github/pull_request_template.md)

总结一下，一个插件应该指定适用于什么环境（vite 还是 rollup），开源的，拥有完善文档的，开启你的插件开发之路吧