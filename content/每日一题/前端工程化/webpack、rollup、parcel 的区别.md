---
noteId: 1727841699505
---

# webpack、rollup、parcel 的区别
---
- Webpack
	- **配置**: 相对复杂
	- **场景**: 
		- 大型项目
		- 复杂构建流程的项目
		- 高度定制化的项目
		- 需要处理多种资源类型的项目
	- **Tree Shaking**: 相比 Rollup 没有那么高效
	- **模块化**: 支持 ESM、CommonJS、 AMD、UMD
- Rollup
	- **配置**: 相对简单
	- **场景**: 
		- 库
		- 组件
	- **Tree Shaking**: 相比 Webpack 更高效, 打包体积更小
	- **模块化**: 只支持 ESM
- Parcel
	- **配置**: 几乎为零
	- **场景**: 
		- 中小型项目
		- 需要快速启动的零配置项目
		- 需要快速迭代的原型项目
	- **模块化**: 支持 ESM、CommonJS、 AMD、UMD
