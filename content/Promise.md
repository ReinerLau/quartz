---
tags:
  - javascript
---
# Promise

## 定义

异步编程回调地狱的解决方案

## 特性

- 状态

   - 种类

      - Pending：进行中

      - Resolved：已完成

      - Rejected：已拒绝

   - 变更

      - 通过 resolve 和 reject 函数变更状态

      - 状态变更后无法二次变更

- 方法

   - then

   - catch

   - finally

   - all

   - race

- 初始化实例时的回调是立即执行的