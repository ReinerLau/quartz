---
tags:
  - bug
---
# 原因

这是 ios 上对可点击的元素的默认触摸反馈效果, 会在用户点击时显示一个短暂的阴影或高亮效果, 主要目的是在用户与应用程序进行交互时，提供视觉上的即时反馈

# 解决

添加样式

```css
* {
  -webkit-tap-highlight-color: transparent
}
```