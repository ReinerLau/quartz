---
title: 每日一题之 Vue
tags:
  - vue
---
## 响应式原理

- **数据劫持**: 使用 Proxy 或 Object.defineProperty 拦截数据的读写操作, 数据变成响应式数据
- **依赖收集**: 响应式数据被访问时, 收集访问该数据的依赖, 比如组件的渲染函数
- **触发更新**: 响应式数据被修改时, 触发被收集的依赖, 然后重新渲染组件

## MVVM、MVC、MVP 的区别

- MVC:
	- 组成
		- Model: 
			- **数据管理**: 存储、更新数据
			- 通知 View 更新
		- View
			- 显示界面
			- 更新界面
			- 捕捉用户输入
		- Controller
			- **处理业务逻辑**: 决定如何通知 Model 更新数据
	- **通信流程**: 单向通信 
		- View 捕捉用户输入后通知 Controller
		- Controller 处理用户输入后调用 Model 更新数据
		- Model 通知 View 更新
	- **缺点**: View 和 Model 耦合
- MVP:
	- 组成
		- Model 
			- **数据管理**: 存储、更新数据
			- 返回数据给 Presenter
		- View
			- 显示界面
			- 捕捉用户输入
		- Presenter
			- **处理业务逻辑**: 决定如何通知 Model 更新数据
			- 更新 View
	- 通信流程
		- View 捕捉到用户输入后通知 Presenter
		- Presenter 处理用户输入后调用 Model 更新数据
		- Model 返回数据给 Presenter
		- Presenter 根据返回数据更新 View
	- **优点**: View 和 Model 之间没有直接联系
	- **缺点**: Presenter 可能会变得臃肿
- MVVM
	- 组成
		- Model
			- **数据管理**: 存储、更新数据
		- View
			- 显示界面
			- 捕捉用户输入
			- **双向绑定**: 和 ViewModel 自动同步数据
		- ViewModel
			- **处理业务逻辑**: 负责将 Model 中的数据转换给 View 使用
			- **双向绑定**: 和 View 自动同步数据
	- 通信流程
		- View 捕捉到用户输入后通知 ViewModel
		- ViewModel 处理用户输入后调用 Model 更新数据
		- Model 返回数据给 ViewModel
		- ViewModel 转换数据后自动同步到 View 上

## Computed 和 Watch 的区别

- computed
	- **依赖**: 依赖的数据发生变化自动重新计算
	- **缓存**: 缓存计算后的结果, 多次使用时不会每次都重新计算
	- **场景**: 计算派生值
- watch
	- **监听**: 监听数据的变化并执行自定义的回调函数
	- **场景**: 某个数据变化后触发额外操作, 产生副作用

## Computed 和 Methods 的区别

- computed: [[每日一题之 Vue#Computed 和 Watch 的区别|参考]]
- methods: 每次使用都会执行, 不会缓存

## v-if 和 v-show 的区别

- v-if
	- **渲染**: 条件为真则添加到 DOM, 条件为假则从 DOM 移除
	- **场景**: 条件不常改变, 初始状态不需要渲染的元素
- v-show
	- **渲染**: 通过 `display` 控制显隐
	- **场景**: 需要频繁切换显隐

## data 为什么是一个函数而不是对象

vue 组件通常是可以复用的 - 每次使用组件都会创建新的组件实例

如果 `data` 是对象 - 每个组件实例共享同一个 `data` 对象 - 一个组件实例修改 `data` 会影响其他组件实例

如果 `data` 是函数 - 每个组件实例返回一个全新的对象 - 每个组件实例互不影响

## 单页应用与多页应用的区别

- 单页应用
	- **加载**: 只有一个 HTML 文件, 页面导航时通过请求动态加载内容, 页面不会刷新
	- **体验**: 不需要刷新, 页面切换更流畅
	- **SEO**: 支持不好, 需要服务端渲染支持
- 多页应用
	- **加载**: 每个页面都有单独的 HTML 文件, 页面导航时会刷新
	- **体验**: 每次切换页面都要重新加载资源, 有明显延迟
	- **SEO**: 支持较好




