---
title: 什么是 monorepo
tags:
  - 工程化
---
## 为什么需要 monorepo

### multirepo

与 monorepo 对应的就是 multirepo，即多个仓库分别管理多个模块，比如一个项目用一个仓库管理，多个项目共同依赖的包也用一个仓库管理，优点就是每个模块可以独立开发和部署。多个项目想要共用一个包，需要将包发布到 npm 仓库上，然后每个项目安装该包，假设包有 bug 需要更新，则需要走以下流程：

- 提交修复到包的仓库上
- 对包进行打包构建
- 发布新版本的包到 npm 仓库上
- 项目 A 更新包版本
- 项目 B 更新包版本
- 项目 A 打包构建部署
- 项目 B 打包构建部署

如果越来越多项目依赖该包，这个流程就会更长更繁琐

### monorepo

即一个仓库管理多个模块，和单纯地把多个模块堆在一个仓库里不同的是，每个模块单独管理对应需要的依赖，即每个模块对应一个 package.json 文件，也就是每个模块有对应的边界

对于多个项目共同依赖的包，同样每个项目也要安装该包，不同的是，如果包出现更新，不需要分别更新每个项目对应的包版本，因为不依赖 npm 仓库上的版本，而是依赖于本地的版本，其流程如下：

- 提交修复到仓库上
- 项目 A 打包构建部署
- 项目 B 打包构建部署

That‘s it！大大加速了迭代流程

## 搭建

搭建方式主要有两种

- Lerna + yarn workspace
- pnpm

pnpm 相对简单一点，和 npm、yarn 一样都是包管理器，所以先采用这个方式实践

全局安装 pnpm

```shell
npm install pnpm -g
```

检查是否安装成功

```shell
pnpm -v
```

创建项目目录结构，`packages` 下是每一个包目录

```shell
.
└── monorepo-demo/
    └── packages/
        ├── pkg1
        ├── pkg2
        └── shared
```

在根目录和每个包目录下使用 `pnpm init` 初始化 package.json 文件，初始化后目录如下

```shell
.
└── monorepo-demo/
    ├── packages/
    │   ├── pkg1/
    │   │   └── package.json
    │   ├── pkg2/
    │   │   └── package.json
    │   └── shared/
    │       └── package.json
    └── package.json
```

根目录下创建 `pnpm-workspace.yaml` 文件，表示工作区中包含哪些包

```yml
packages:
  - "packages/*"
```

最终目录

```shell
.
└── monorepo-demo/
    ├── packages/
    │   ├── pkg1/
    │   │   └── package.json
    │   ├── pkg2/
    │   │   └── package.json
    │   └── shared/
    │       └── package.json
    ├── package.json
    └── pnpm-workspace.yaml
```

至此，项目就算搭建完成了

## 安装依赖

项目依赖分为两种，一种是公共依赖，另一种是特有依赖

### 公共依赖

指的是所有包共享的依赖，比如开发规范配置，`eslint`、`prettier`、`typescript` 等

在根目录下安装依赖, `-w` 明确告诉 pnpm 安装依赖到根目录下，并更新根目录的 package.json

```shell
pnpm add eslint --save-dev -w
```

### 特有依赖

指的是某个包依赖本地另一个包或来自 npm 的包

比如 `pkg1` 安装 npm 上的 `lodash`，使用 `--filter` 指定安装到本地哪个包，名字和 package.json 中的 `name` 字段对应

```shell
pnpm add lodash --filter pkg1
```

比如 `pkg1` 安装本地的 `pkg2`

```shell
pnpm add pkg2 --filter pkg1
```

要注意安装本地包后，package.json 对应包的版本号带有 `workspace` 字眼，表示来自本地工作区定义的包，否则来自 npm 仓库上
