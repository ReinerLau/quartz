---
title: 关于 Tag
tags:
  - github
---
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

# 规范

1. 开头加上 v 前缀（v1.0.0, v2.3.4）
2. 预发布版本加上后缀（v0.2.0-alpha, v5.9-beta.3）
3. [更多](https://semver.org/)