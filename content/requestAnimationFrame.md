---
tags:
  - javascript
---
# requestAnimationFrame

## 定义

将回调函数延迟到下一帧重绘之前执行

## 原理

- 浏览器按照一定频率进行重绘

   - 频率和屏幕刷新率保持一致

   - 每秒 60 次 = 每 1000 毫秒 60 次 = 每 16.7 毫秒重绘一次 = 每 16.7 毫秒产生一帧动画 = 每一帧回调必须在 16.7 毫秒内执行完成

## 拓展

- 场景

   - 上一个 requestAnimationFrame 回调里调用了下一个 requestAnimationFrame

      - 上一个回调和重绘在一帧内完成：下一个回调延迟到下一帧执行

      - 上一个回调和重绘无法在一帧内完成：下一个回调马上执行

   - 同时执行多个 requestAnimationFrame：下一帧会依次执行多个回调再一次性重绘