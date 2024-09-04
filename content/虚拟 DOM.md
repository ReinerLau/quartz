# 虚拟 DOM

## 定义

使用 JS 对象模拟真实 DOM 节点

## 特性

- 降低性能开销

   - 避免引用占用大内存的真实 DOM 节点

   - 结合 diff 算法避免视图更新时不必要的 DOM 操作

- 提供跨平台能力：抽象了渲染过程

## 拓展

性能：不一定比真实 DOM 性能好，因为多了转换过程

---

<https://vue3js.cn/interview/vue/vnode.html>