---
tags:
  - css
title: 定位
---
# 什么是盒模型

- **定义**: 所有元素都可以看作是一个“盒子”
- 组成
	- content
	- padding
	- border
	- margin
- 类型
	- 标准盒模型
		- **默认**：所有元素默认都是标准盒模型
		- **宽高组成**：只包含 content
		- **设置**：通过 `box-sizing: content-box` 设置
	- 怪异盒模型
		- **宽高组成**：包括 content、padding、border
		- **设置**：通过 `box-sizing: border-box` 设置

# 如何实现清除浮动

- clearfix 类
   - :after 伪类
   - 空白元素
   - clear:both
- [[每日一题之 CSS#什么是 BFC|BFC]]

# 如何实现水平垂直居中

- 绝对定位 + translate
- 绝对定位 + margin
- [[每日一题之 CSS#弹性布局|弹性布局]]

# 如何实现响应式布局

- [[每日一题之 CSS#什么是媒体查询|媒体查询]]
- 百分比
- rem: 根据根元素 font-size 决定子元素大小的单位
- 视口单位
	- vw：[[视觉视口]]宽度的百分比
	- vh：[[视觉视口]]高度的百分比
	- vmin：[[视觉视口]]宽度与[[视觉视口]]高度间的较小值
	- vmax：[[视觉视口]]宽度与[[视觉视口]]高度间的较大值
- [[每日一题之 CSS]]
- 网格布局
- UI 库栅格布局

# 说一下选择器的优先级

- 100
	- id
- 10
	- 类
	- 伪类
	- 属性
- 1
	- 标签
	- 伪元素
- 0
	- 相邻兄弟
	- 子
	- 后代
	- 通配符

# 什么是 BFC

- **定义**: 内部元素不会影响外部元素布局的区域
- 特性
	- 浮动元素
		- 计算高度时会包含区域内浮动元素
		- 不会与区域外浮动元素发生重叠
	- 垂直外边距合并
		- 区域内垂直相邻元素会发生垂直外边距合并
		- 不会与区域外垂直相邻元素发生垂直外边距合并
- 创建
	- 元素设置浮动
	- 元素设置[[每日一题之 CSS#都有哪几种定位方式|绝对定位]]
	- overflow
		- hidden
		- auto
		- scroll
	- display
	    - inline-block
	    - flex

# 什么是设备像素比

- **定义**: 设备独立像素与设备像素的转换关系, 简称 dpr
- **作用**: 解决相同设备独立像素在不同设备像素下的效果差异问题
- **获取**: window.devicePixelRation
- **缩放**: PC 端缩放网页会修改该值 
- **设备独立像素**: 与设备无关的逻辑像素, 比如 CSS
- **设备像素**: 设备屏幕上的物理像素

# 什么是媒体查询

通过识别设备类型和参数应用对应样式

# 都有哪几种定位方式

- **static**（默认）
	- 没有定位
	- 忽略 `top`、`bottom`、`right`、`left`、`z-index` 属性
- **relative**
	- 相对于原来位置进行定位
	- 通过 `top`、`bottom`、`right`、`left` 设置位置
- **absolute**
	- 相对于最近非 `static` 定位的父元素进行定位
	- 脱离文档流
	- 通过 `top`、`bottom`、`right`、 `left` 设置位置
- **fixed**
	 - 相对于[[视觉视口]]进行定位
	 - 脱离文档流
	 - 通过 `top`、`bottom`、`right`、`left` 设置位置
	
# 弹性布局

- 组成
    - 容器
    - 项目
    - 主轴
    - 交叉轴
- 容器属性
    - flex-flow
        - flex-direction：主轴方向
        - flex-wrap：换行方式
    - 对齐
        - justify-content：主轴对齐方式
        - align-items：交叉轴对齐方式
        - align-content：多根轴线对齐方式
- 项目属性
    - order：项目排列顺序
    - flex
        - flex-grow：项目占用剩余空间比例
        - flex-shrink：项目吸收溢出空间比例
        - flex-basis：项目占用剩余空间的初始大小
    - align-self：项目自身对齐方式

# display

- none：脱离文档流
---
- block：变成[[每日一题之 CSS#块级元素、行内元素和行内块级元素的区别|块级元素]]
- inline：变成[[每日一题之 CSS#块级元素、行内元素和行内块级元素的区别|行内元素]]
- inline-block：变成[[每日一题之 CSS#块级元素、行内元素和行内块级元素的区别|行内块级元素]]
---
- flex：变成[[每日一题之 CSS#弹性布局|弹性布局]]的容器
- grid：变成网格布局的容器