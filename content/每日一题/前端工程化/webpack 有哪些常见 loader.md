---
noteId: 1727841699380
---


# webpack 有哪些常见 loader
---
- babel-loader
	- **兼容**: 将现代 JavaScript 代码转换为后兼容的 JavaScript 代码
- css-loader
	- **解析**: 递归解析 CSS 文件中的 `@import` 语句和 `url()` 函数
	- **转换**: 将解析到的 CSS 文件转换为JavaScript 模块, 通过 `import` 引入, 导出一个包含 CSS 类名映射的对象
- style-loader
	- **注入**: 将 CSS 注入到 `<style>` 标签中
	- **css-loader**: 通常与 `css-loader` 一起使用
- file-loader
	- **复制**: 将文件复制到输出目录，
	- **URL**: 并返回文件的 URL 给代码动态引用
- url-loader
	- **file-loader**: 类似于 file-loader
	- **转换**: 如果文件小于指定大小，则将其转换为 Base64 URL
- sass-loader
	- **编译**: 将 Sass 或 SCSS 文件编译为 CSS
- less-loader
	- **编译**: 将 Less 文件编译为 CSS