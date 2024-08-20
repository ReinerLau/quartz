# 如何实现 Mock

## vite-plugin-mock

通过阅读 vben 的官方文档可以得知项目使用了一个 vite 插件 [vite-plugin-mock](https://github.com/vbenjs/vite-plugin-mock) 来实现，那首先先看看这个插件的基本用法

粗略阅读这个仓库，可以得知几点信息：

- 插件由 vben 团队开发，也就代表说和本项目总体来说是兼容的
- 支持本地环境和生产环境的 Mock
- 本地环境 Mock 通过 Connect 服务中间件实现，从 vben 官方文档可以得知这个 Node.js 中间件应该是通过参数拦截实现的，这样就可以在本地浏览器控制台查看网络请求记录
- 生产环境 Mock 通过 Mockjs 这个比较常见的库实现

### 基本用法

#### 配置

```shell
pnpm add --save-dev vite-plugin-mock
```

`vite.config.ts`

```ts
export default defineConfig({
  plugins: [
    viteMockServe({
      mockPath: "mock",
      enable: true,
    }),
  ],
});
```

`mockPath` 表示存放 mock 文件的的位置

`enable` 表示开启 mock 功能

#### mock 文件

```ts
import { MockMethod } from "vite-plugin-mock";
export default [
  {
    url: "/api/get",
    method: "get",
    response: ({ query }) => {
      return {
        code: 0,
        data: {
          name: query.name,
        },
      };
    },
  },
] as MockMethod[];
```

接下来发送请求就会自动匹配接口地址并拦截返回模拟的数据

## 排除 `_` 开头文件夹及文件

vben 官方文档看到一句 Tip

> 文件新增后会自动更新，不需要手动重启，可以在代码控制台查看日志信息 mock 文件夹内会自动注册，排除以 `_` 开头的文件夹及文件

这里有两个疑问：

- 怎么排除？
- `_` 开头的文件夹及文件是什么？

### 怎么排除？

从插件的文档和 vben 项目的配置得知使用 `ignore` 选项可以忽略对指定文件的读取，防止同一 `mockPath` 下的文件自动作为 mock 文件被读取

vben 相关的插件配置是这样的

```ts
import { viteMockServe } from "vite-plugin-mock";

export function configMockPlugin() {
  return viteMockServe({
    ignore: /^_/,
    mockPath: "mock",
  });
}
```

### `_` 开头的文件夹及文件是什么？

包括一些可以复用的函数和工具函数

## mockjs 语法

从 vben 文档看到一个 Tip

> mock 的值可以直接使用 mockjs 的语法。

例如下面的 mock 文件

```ts
import { MockMethod } from "vite-plugin-mock";
import { resultPageSuccess } from "../_util";

const demoList = (() => {
  const result: any[] = [];
  for (let index = 0; index < 60; index++) {
    result.push({
      id: `${index}`,
      beginTime: "@datetime",
      endTime: "@datetime",
      address: "@city()",
      name: "@cname()",
      "no|100000-10000000": 100000,
      "status|1": ["正常", "启用", "停用"],
    });
  }
  return result;
})();

export default [
  {
    url: "/api/table/getDemoList",
    timeout: 1000,
    method: "get",
    response: ({ query }) => {
      const { page = 1, pageSize = 20 } = query;
      return resultPageSuccess(page, pageSize, demoList);
    },
  },
] as MockMethod[];
```

但插件文档中却找不到相关说明，唯一能说明的应该只有“mockjs 在生产环境中使用”这句话，这里可以提一个 issue，而且不管安不安装 mockjs 依赖都能在开发环境使用该语法

## 去掉 mock

当后台接口已经开发完成就不需要 mock，去掉的方式，vben 文档介绍的只有一种，注释掉或删除掉对应 mock 接口

## 线上 mock

按照插件文档，只要调用 `createProdMockServer` 函数并传入所有 mock 模块就可以，在 vben 文档中则通过插件选项的 `injectCode` 注入代码实现调用，因为即使通过在 `main.ts` 中判断是否调用，在关闭了 mock 功能的情况下也会将 `mockjs` 打包到构建文件中，导致增加打包体积

但是插件文档没有 `injectCode` 的说明，这也是一个 issue

## 完整配置

```ts
import { viteMockServe } from "vite-plugin-mock";

export function configMockPlugin({ isBuild }: { isBuild: boolean }) {
  return viteMockServe({
    ignore: /^_/,
    mockPath: "mock",
    localEnabled: !isBuild,
    prodEnabled: isBuild,
    injectCode: `
      import { setupProdMockServer } from '../mock/_createProductionServer';

      setupProdMockServer();
      `,
  });
}
```

`ignore` 忽略掉对 `_` 开头文件夹及文件的读取

`mockPath` 读取 `mock` 文件夹下的 mock 模块

`localEnabled` 猜测是开发环境下是否开启 mock 功能的意思

`prodEnabled` 则是生产环境下是否开启 mock 功能的意思

`injectCode` 测试是开启了 prodEnabled 时才会注入代码

`isBuild` 表示是否正在运行构建命令

## 整体架构

```sh
.
├── internal/ # 项目内部使用的库
│   └── vite-config/ # vite 配置
│       └── src/ # 主目录
│           └── plugins/ # vite 插件相关
│               └── mock.ts # mock 插件相关
└── mock/ # mock 相关
    ├── demo # mock 模块
    ├── sys # mock 模块
    ├── _createProductionServer.ts # 线上 mock
    └── _utils # 工具函数
```

## 参考

- https://doc.vvbin.cn/guide/mock.html#mock-%E6%9C%8D%E5%8A%A1
- https://github.com/vbenjs/vite-plugin-mock/blob/main/README.zh_CN.md
