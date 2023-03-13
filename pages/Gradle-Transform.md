- ## 一、概念
  collapsed:: true
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
- ## 二、Transform使用场景
  collapsed:: true
	- 对编译class文件做自定义的处理
	- 读取编译产生的class文件，做一些其他事情，但是不需要修改它。
- ## 三、执行流程：
  collapsed:: true
	- ![image.png](../assets/image_1678694029928_0.png)
- ## 四、难点：
	- transform 的核心难点有以下几个点：
	- 正确、高效的进行文件目录、jar 文件的解压、class 文件 IO 流的处理，保证在这个过程中不丢失文件和错误的写入
	- 高效的找到要插桩的结点，过滤掉无效的 class
	- 支持增量编译
- ## 五、API
	- ### getName()
		- 指明 Transform 的名字，也对应了该 Transform 所代表的 Task 名称
		- 举例：
		  collapsed:: true
			- ```kotlin
			  class UnityLogTransform : Transform() {
			      override fun getName(): String {
			          return "UnityLogTransform"
			      }
			  }
			  ```
	- ### getInputTypes()
		- getInputTypes()方法用于返回Transform所支持的输入文件类型，开发者可以通过返回这些常量中的任意一个来指定Transform的输入文件类型。
		- 取值：
		  collapsed:: true
			- 在Android Gradle插件 4.0及以上的版本中，TransformManager.CONTENT_CLASS、TransformManager.CONTENT_JARS等常量已经被废弃，取而代之的是QualifiedContent.DefaultContentType类中的一些常量。例如，
			- QualifiedContent.DefaultContentType.CLASSES表示class文件类型，
			- QualifiedContent.DefaultContentType.RESOURCES表示资源文件类型
		- 举例：
		  collapsed:: true
			- ```kotlin
			      override fun getInputTypes(): MutableSet<QualifiedContent.ContentType> {
			          //QualifiedContent.DefaultContentType.RESOURCES 支持资源输入类型
			          //QualifiedContent.DefaultContentType.CLASSES   支持class类型
			         return mutableSetOf(QualifiedContent.DefaultContentType.CLASSES)
			      }
			  ```
	- ### getScopes()
		- 是Transform类中的一个方法，用于返回Transform所处理的文件的范围。它返回一个Set对象，包含了Transform所支持的所有文件范围。
		- 取值：
		  collapsed:: true
			- Scope.PROJECT:
				- 表示Transform所处理的是项目本身的文件。
			- Scope.SUB_PROJECTS:
				- 表示Transform所处理的是项目下的所有子项目的文件。
			- Scope.EXTERNAL_LIBRARIES:
				- 表示Transform所处理的是项目依赖的所有外部库的文件。
			- Scope.PROVIDED_ONLY:
				- 表示Transform所处理的是项目中provided配置下的依赖库的文件。
			- Scope.TESTED_CODE:
				- 表示Transform所处理的是测试代码的文件。
		- 示例：
		  collapsed:: true
			- ```kotlin
			  
			      override fun getScopes(): MutableSet<in QualifiedContent.Scope> {
			          return mutableSetOf(
			              QualifiedContent.Scope.PROJECT, 
			              QualifiedContent.Scope.SUB_PROJECTS,
			              QualifiedContent.Scope.EXTERNAL_LIBRARIES
			          )
			      }
			  ```
	- ### isIncremental()
		- 表明是否支持增量编译，不是每次的编译都可以增量编译，clean build没有增量的基础，需要检查当前的编译是否增量编译。
			-
-
-
-
-
-
- 参考：
  collapsed:: true
	- [刚学会Transform，你告诉我就要被移除了](https://juejin.cn/post/7114863832954044446)
-