---
title: Mock
tags:
  - 工程化
---
## 为什么需要 Mock

正常的开发流程中，前后端会约定好接口，在后端没开发好接口之前，前端的开发进度会被阻塞，这时候就需要 Mock，通过模拟接口数据，支持前端独立开发，待后端接口开发完成再实现替换

## vite-plugin-mock

这个 vite 插件和 Mockjs 不同的是，支持在开发环境下拦截请求，实现在开发者工具中查看网络请求记录

### 基本用法

安装依赖

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

在 `mockPath` 文件路径下新建 mock 文件

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

### mockjs 语法

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


### 线上 mock

按照插件文档，只要调用 `createProdMockServer` 函数并传入所有 mock 模块就可以，在 vben 文档中则通过插件选项的 `injectCode` 注入代码实现调用，因为即使通过在 `main.ts` 中判断是否调用，在关闭了 mock 功能的情况下也会将 `mockjs` 打包到构建文件中，导致增加打包体积

但是插件文档没有 `injectCode` 的说明，这也是一个 issue