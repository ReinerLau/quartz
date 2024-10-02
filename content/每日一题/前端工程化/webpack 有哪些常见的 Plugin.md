---
noteId: 1727841699404
---


# webpack 有哪些常见的 Plugin
---
- HtmlWebpackPlugin
	- **生成**: 自动生成 HTML 文件
	- **注入**: 注入打包后的 JavaScript 和 CSS 文件到生成的 HTML 文件中
- CleanWebpackPlugin
	- **清理**: 在每次构建之前清理输出目录
- MiniCssExtractPlugin
	- **提取**: 将内嵌在 JavaScript 中的 CSS 代码提取出来, 生成独立的 CSS 文件
- TerserWebpackPlugin
	- **压缩**: 压缩 JavaScript 代码
- DefinePlugin
	- **注入**: 允许在编译时将变量注入到代码中, 比如定义全局常量，可以在开发和生产环境中使用不同的配置
- HotModuleReplacementPlugin
	- **热更新**: 启用模块热替换（HMR）
- BundleAnalyzerPlugin
	- **分析**:  分析和可视化打包后的文件大小