- ## 一、什么是Transform
	- 打包流程：
	  collapsed:: true
		- ![image.png](../assets/image_1678693760387_0.png)
	- ### 什么是transform?
		- AGP包含了一个Transform API, 它允许第三方插件在编译后的class文件转换为dex文件之前做处理操作. 而使用Transform API, 我们完全可以不用去关注相关task的生成与执行流程, 我们可以只聚焦在如何对输入的class文件进行处理
	- ### 工作时机 ：
	  collapsed:: true
		- Transform工作在Android构建中由Class → Dex 的节点
	- ### 处理对象：
	  collapsed:: true
		- 处理对象包括Javac编译后的Class文件、Java标准 resource 资源、本地依赖和远程依赖的 JAR/AAR
	- ### Transform Task：
	  collapsed:: true
		- 每个Transform都对应一个Task，Transform 的输入和输出可以理解为对应 Transform Task 的输入输出
	- ### Transform 链：
	  collapsed:: true
		- TaskManager 会将每个TransformTask 串联起来，前一个Transform的输出会作为下一个 Transform 的输入
	-