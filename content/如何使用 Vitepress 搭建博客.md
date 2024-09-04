---
title: 如何使用 Vitepress 搭建博客
tags:
---
## 安装依赖

```shell
pnpm add --save-dev vitepress
```

## 初始化

```shell
pnpm vitepress init
```

`Where should VitePress initialize the config` 输出当前目录 `./`

其他选项随意

## 运行

```shell
pnpm docs:dev
```

## 部署

因为站点部署在 `https://reinerlau.github.io/blog` 子路径，但 html 文件获取资源时请求的是根路径，需要在配置中设置 `base` 选项为 `/blog/`

在仓库设置中的`Pages`菜单项下，选择`Build and deployment > Source > GitHub Actions`

在 `.github/workflows` 下创建 Github Action 的工作流配置文件 `deploy.yml`

```yml
name: Deploy VitePress site to Pages

on:
  push:
    branches: [main]

  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: pnpm/action-setup@v3
        with:
          version: 9
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
      - name: Setup Pages
        uses: actions/configure-pages@v4
      - name: Install dependencies
        run: pnpm install
      - name: Build with VitePress
        run: pnpm docs:build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: .vitepress/dist
  deploy:
    environment: 
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    needs: build
    runs-on: ubuntu-latest
    name: Deploy
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

每次推送代码到 `main` 分支上就会自动执行 Github Action 工作流并自动部署，等待完成后即可访问 `https://reinerlau.github.io/blog`

## 参考

- https://vitepress.dev/zh/guide/getting-started
- https://vitepress.dev/zh/guide/deploy#github-pages