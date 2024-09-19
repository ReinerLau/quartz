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