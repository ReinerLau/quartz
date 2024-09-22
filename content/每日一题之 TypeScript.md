---
tags:
  - typescript
---
# unknow 和 never 的区别

- never
	- **含义**: 永远不会有值
	- **场景**: 函数永远不会返回, 如死循环、抛出异常
- unknown
	- **含义**: 不确定的类型, 比 any 更严格
	- **场景**: 判断类型后再使用变量

# TypeScript 的优点

- 静态分析
- IDE 补全
- 代码重构
- 代码文档

# TypeScript 的特性

- interface
- type
- 枚举
- 泛型
- 类型推断
- 类型断言

# type 和 interface 的区别

- type
	- **表示类型**: 能够表示非对象类型, 比如函数
	- **合并**: 支持类型的合并
	- **场景**: 涉及到复杂运算
- interface
	- **表示类型**: 只能表示对象类型
	- **继承**: 支持继承其他 interface
	- **合并**: 自动合并同名 interface