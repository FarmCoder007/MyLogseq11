- # 设计
	- 1. 找一个复杂的 SDK 进行手动改造，带 demo 的，比如 WubaHybridSDK，改造完后，可以正常编译运行 demo
	- 2. 代码变动梳理：
	    2.1 代码变动：demo、library
	    2.2 依赖变动
	    2.3 构建脚本变动：gradle、gradle plugin、kotlin、gradle.properties(androidX)...
	    2.4 资源布局等变动：比如 layout 中引用的 support 库、RecyclerView..
	    2.5 混淆变动
	- 3. 脚本设计
	    3.1 python/shell migrate 6.1 inputPath outputPath
	    3.2 脚本中配置 6.1 的目标版本号：gradle、gradle plugin、kotlin、第三方库版本号
	    3.3 进行处理
	    3.4 输出报告：
	        3.4.1 按 module 输出变更的依赖映射
	        3.4.2 按 module 输出变更的代码映射
	        3.4.3 按 module 输出变更的布局映射
	        3.4.4 变更的混淆配置
- # 变动
- # 脚本设计
	- 1、