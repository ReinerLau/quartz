---
title: 关于 Actions
tags:
  - github
---
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

