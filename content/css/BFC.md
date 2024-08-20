---
tags:
  - css
---
# BFC

## 定义

内部元素不会影响外部元素布局的区域

## 特性

- 浮动元素

   - 计算高度时会包含区域内浮动元素

   - 不会与区域外浮动元素发生重叠

- 垂直外边距合并

   - 区域内垂直相邻元素会发生垂直外边距合并

   - 不会与区域外垂直相邻元素发生垂直外边距合并

## 拓展

- 创建

   - 根元素

   - 元素设置浮动

   - 元素设置绝对定位

   - overflow

      - hidden

      - auto

      - scroll

   - display

      - inline-block

      - flex

---

- <https://juejin.cn/post/7303392509664247835>

- <https://juejin.cn/post/6844903504545316877?searchId=202401051002443C199A7A109E9B6DD518#heading-5>