# 该博客是怎么炼成的

## 初始化

克隆 Quartz 仓库

```shell
git clone https://github.com/jackyzha0/quartz.git
```

把 `quartz` 文件夹名改成自己想要的名字比如 `blog`

安装依赖（用淘宝镜像，否则贼慢）

```shell
npm install
```

初始化 Quartz

```shell
npx quartz create
```

分别选择 `Empty Quartz` 和 `Treat links as shortest path`

然后运行在 http://localhost:8080 预览

```shell
npx quartz build --serve
```

## 部署

先删除克隆项目的 `.git` 目录

推送代码到自己的仓库上

创建 `.github/workflows/deploy.yml`

```yml
name: Deploy Quartz site to GitHub Pages

on:
  push:
    branches:
      - main

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Fetch all history for git info
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - name: Install Dependencies
        run: npm ci
      - name: Build Quartz
        run: npx quartz build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: public

  deploy:
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

仓库进入 `Settings - Pages - Source` 设置为 `Github Action`

现在推送代码就能自动部署到 Github Page 上

## 使用 Obsidian 编辑

直接用 Obsidian 打开项目下的 `content` 目录即可
