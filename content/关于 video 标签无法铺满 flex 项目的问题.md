---
tags:
  - bug
---
# 原因

还在摸索中...

# 解决

`<video>` 添加样式

```css
video {
    position: absolute;
    left: 0;
    top: 0;
    height: 100%;
    width: 100%;
}
```

父元素设置为相对定位

# 参考

- https://stackoverflow.com/questions/34835042/scale-video-to-fit-inside-flexbox-item-while-keeping-aspect-ratio