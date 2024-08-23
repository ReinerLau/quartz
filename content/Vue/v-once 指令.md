---
tags:
  - vue
---
# v-once 指令

## 定义

仅渲染元素和组件一次并跳过之后的更新

## 原理

- 静态提升

   1. 初始渲染

   2. 标记成静态节点

   3. 缓存静态节点

   4. 更新

   5. 复用缓存中的静态节点

---

<https://cn.vuejs.org/guide/extras/rendering-mechanism#static-hoisting>