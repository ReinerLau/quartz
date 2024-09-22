---
title: 关于 Tag
tags:
  - github
---
# Actions
## 作用

- 执行自定义 CI/CD 流程
- 使用别人写好的流程 

## 概念

- workflow：工作流程，持续集成一次运行的过程。
- job：任务，一个 workflow 由一个或多个 job 构成，含义是一次持续集成的运行，可以完成多个任务。
- step：步骤，每个 job 由多个 step 构成，一步步完成。
- action：动作，每个 step 可以依次执行一个或多个动作。

## 使用

在 `.github/workflows` 文件夹下创建 `*.yml` 文件

示例
```yml
name: "test"
on: "push"
jobs:
  test_job:
    name: "test job"
    runs-on: "ubuntu-latest"
    steps:
      - name: Checkout
        uses: actions/[checkout@master](mailto:checkout@master)
      - name: print
        run: |
          echo "hello world"
```

- `name`：可选，作为查看流程时的名字，省略则使用 yml 文件的名字
- `on`：监听哪个事件被触发后执行流程，包括代码库事件、外部事件、定时触发
- `jobs`：每个 `job` 有一个 `id`，也可以使用 `name` 修改显示的名字
- `runs-on`：运行的环境
- `steps`: 每个 `step` 用 `-` 分开，`uses` 使用别人写好的 `action`

## VSCode 插件

这个[插件](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-github-actions)可以支持在 vscode 中查看 workflow 的运行状况，而不用跑到网页上查看

![image](https://github.com/ReinerLau/github-guide/assets/103234074/862bbab5-6f12-4d7a-a26c-9c935d1adb75)

# Github Pages

创建`用户名.github.io`仓库，用户名小写

选择想要部署的仓库，选择要部署的分支，选择 index.html、index.md、README.md 所在文件夹
![image](https://github.com/ReinerLau/github-guide/assets/103234074/979c771c-e0a4-410b-bc87-bfa64c28fa89)

保存之后自动执行 github action workflow（保存按钮灰色就先将分支切到 None 再切回其他分支）
![image](https://github.com/ReinerLau/github-guide/assets/103234074/5eafb328-fbd8-43bd-a9b6-eedd03e35dc4)

执行完看到地址
![image](https://github.com/ReinerLau/github-guide/assets/103234074/8d82104c-3c4f-425f-8db7-26406254469f)
![image](https://github.com/ReinerLau/github-guide/assets/103234074/da3a0690-f11c-4125-a3e1-57fc1ba2b398)

# License

## 创建

从[choosealicense](https://choosealicense.com/)里选择一个 license 并复制

在项目根目录下创建 LICENSE.txt、LICENSE.md 或 LICENSE.rst 文件并粘贴复制的内容

推送之后就能在 github 仓库右侧和 README 右侧看到

# Release

## 和 tag 的区别

- 属于 github 的概念不是 git 的概念
- 基于 tag 创建，不是每个 tag 都有对应的 release
- 可以包含 changelog 信息
- 可以包含当前 tag 历史版本下编译后的文件
# Tag

## 为什么

- 指向某个 commit
- 标记版本
- 不改动只查看

##  和 Branch 的区别

- 一系列 commit 的组合
- 可以在某个 tag 基础上修改并发布新的 tag

## 创建

前提是安装了 GitLens 插件

### 给当前分支打 tag

![image](https://github.com/ReinerLau/github-features-demo/assets/103234074/74293c2b-1f5b-4396-976d-6802279a46b4)

### 给 commit 打 tag

![image](https://github.com/ReinerLau/github-features-demo/assets/103234074/50f33a8c-22c7-456c-b607-7b113ef209ea)

### 推送 tag

![image](https://github.com/ReinerLau/github-features-demo/assets/103234074/282f48a1-b742-416e-86f0-c6adcb5dff5a)

## 规范

1. 开头加上 v 前缀（v1.0.0, v2.3.4）
2. 预发布版本加上后缀（v0.2.0-alpha, v5.9-beta.3）
3. [更多](https://semver.org/)


