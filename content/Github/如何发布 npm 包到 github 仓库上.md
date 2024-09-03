---
title: 如何发布 npm 包到 github 仓库上
tags:
  - 工程化
  - github
---
## 为什么

发布 npm 包除了发布到[官方的 npm 仓库](https://www.npmjs.com/)上或者自己搭建的[[如何搭建私有 npm 仓库|私有 npm 仓库]], 还有一种比较少见的, 就是发布到自己的 github 仓库上, 只有相关仓库或组织权限的成员才能下载该 npm 包，达到免费私有的目的

## 登录

```sh
npm login --registry=https://npm.pkg.github.com
```

- `username` 小写
- `password` 填 [[如何通过 token 获取 github 仓库权限|token]]

## 初始化包

```sh
npm init
```

- `name` 字段格式为`@用户名小写/包名`

## 指定包发布位置

```json
{
  // ...
  "publishConfig": {
    "registry": "https://npm.pkg.github.com"
  },
  // ...
}
```

# 发布

```shell
npm publish
```

# 设置为公共可见

默认为私有, 进入 `Packages - Package settings - Danger Zone - Change visibility` 可以设置公共可见, 这样所有开发者都可以下载你的包 

# 下载

```shell
npm install @reinerlau/test@1.0.0 --registry=https://npm.pkg.github.com
```

## 参考

https://github.com/ReinerLau/github-guide