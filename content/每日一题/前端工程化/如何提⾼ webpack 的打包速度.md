---
noteId: 1727841699203
---


# 如何提⾼ webpack 的打包速度
---
- 优化 resolve 配置: 通过优化 resolve 配置, 可以减少 Webpack 查找模块的时间
- Tree Shaking: 开启 Tree Shaking, 避免打包不需要的模块
- 缓存
	- cache-loader
	- 持久化缓存
- 多线程处理: 将模块处理任务分配给多个子线程并行处理
	- happypack
	- thread-loader