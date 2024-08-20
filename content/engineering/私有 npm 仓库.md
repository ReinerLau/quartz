---
title: 私有 npm 仓库
tags:
  - 工程化
---
## 为什么需要私有 npm 仓库？

- 团队内部开发的 npm 包只希望内部共享（虽然公共 npm 仓库也支持私有但要收费）
- 避免公共 npm 仓库带来的安全问题，防止恶意代码攻击
- 保证网络稳定，提升下载速度
- 自行管理依赖版本，避免破坏性变更

## Verdaccio

目前最流行的私有仓库搭建工具，相比其他工具，部署简单，同时支持缓存公共 npm 包

### 基本用法

全局安装

```shell
npm install -g verdaccio
```

运行服务

```shell
verdaccio
```

默认服务跑在 http://localhost:4873

从私有仓库中下载包

```shell
npm install --registry http://localhost:4873
```

### 默认仓库

设置发布和下载的默认仓库

```shell
npm set registry http://localhost:4873/
```

### 发布包

```shell
npm publish --registry http://localhost:4873/
```

### 删除包

```shell
npm unpublish pkg --registry http://localhost:4873/
```

删除指定版本

```shell
npm unpublish pkg@1.0.0 --registry http://localhost:4873/
```

### 缓存公共 npm 包

当安装一个私有仓库中没有的 npm 包时，会自动从公共 npm 仓库中下载，下载后缓存起来，下次下载时直接从缓存读取

### docker 部署

```shell
docker run -it --rm --name verdaccio -p 4873:4873 verdaccio/verdaccio
```