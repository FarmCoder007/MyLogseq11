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
	  collapsed:: true
		- 指明 Transform 的名字，也对应了该 Transform 所代表的 Task 名称,Gradle 在编译的时候，会将这个名称经过一些拼接显示在控制台上
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
			- 在Android Gradle插件 4.0及以上的版本中，TransformManager.CONTENT_CLASS、TransformManager.CONTENT_JARS等常量已经被废弃，取而代之的是QualifiedContent.DefaultContentType类中的一些常量。例如，
			- QualifiedContent.DefaultContentType.CLASSES表示class文件类型，Java 字节码文件，
			- QualifiedContent.DefaultContentType.RESOURCES表示资源文件类型,资源，包含 java 文件
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
	  collapsed:: true
		- 表明是否支持增量编译，不是每次的编译都可以增量编译，clean build没有增量的基础，需要检查当前的编译是否增量编译。
		- 不是增量编译，则清空output目录，然后按照前面的方式，逐个class/jar处理
		- 增量编译，则要检查每个文件的Status，Status分为四种，并且对四种文件的操作不尽相同
		  collapsed:: true
			- NOTCHANGED 当前文件不需要处理
			- ADDED、CHANGED 正常处理，输出给下一个任务
			- REMOVED 移除outputProvider获取路径对应的文件
		- 注意：
		  collapsed:: true
			- 增量编译是指在编译过程中，只编译发生变化的文件，而不重新编译所有文件。在Android Gradle插件中，增量编译可以提高构建效率，尤其是在大型项目中。如果一个Transform支持增量编译，Gradle会在执行Transform之前调用isIncremental()方法来查询它是否支持增量编译。如果支持，则在执行Transform时，只处理发生变化的文件。
			- 在Transform的transform方法中，可以通过TransformInvocation的isIncremental()方法来查询增量编译的状态。如果返回值为true，则表示当前执行的是增量编译，Transform只需要处理发生变化的文件。否则，表示当前执行的是全量编译，Transform需要处理所有文件。
			- 增量编译对于一些复杂的Transform可能并不适用，例如需要同时修改多个文件的Transform，因为如果其中一个文件发生了变化，就需要重新处理所有的文件。此时，可以将isIncremental()方法的返回值设为false，禁用增量编译。
		- 示例：
		  collapsed:: true
			- ```kotlin
			  import com.android.build.api.transform.Transform
			  import com.android.build.api.transform.TransformInvocation
			  import com.android.build.gradle.internal.pipeline.TransformManager
			  import com.android.build.api.transform.QualifiedContent
			  import com.android.build.api.transform.TransformOutputProvider
			  import com.android.build.gradle.internal.scope.VariantScope
			  
			  class MyTransform : Transform() {
			  
			      override fun getName(): String {
			          return "myTransform"
			      }
			  
			      override fun getInputTypes(): Set<QualifiedContent.ContentType> {
			          return setOf(QualifiedContent.DefaultContentType.CLASSES)
			      }
			  
			      override fun getScopes(): MutableSet<in QualifiedContent.Scope> {
			          return mutableSetOf(QualifiedContent.Scope.PROJECT)
			      }
			  
			      override fun transform(transformInvocation: TransformInvocation) {
			          if (transformInvocation.isIncremental) {
			              // 处理增量变更的文件
			              transformIncremental(transformInvocation)
			          } else {
			              // 处理所有文件
			              transformFull(transformInvocation)
			          }
			      }
			  
			      override fun isIncremental(): Boolean {
			          return true
			      }
			  
			      private fun transformFull(transformInvocation: TransformInvocation) {
			          // 处理所有文件
			          // ...
			      }
			  
			      private fun transformIncremental(transformInvocation: TransformInvocation) {
			          // 处理增量变更的文件
			          // ...
			      }
			  }
			  
			  ```
	- ### transform()
		- transform是一个空实现，input的内容将会打包成一个 TransformInvocation 对象。
-
- ## 六、自定义transform
	- 实现一个 Transform 需要先创建 Gradle 插件，大致流程：自定义 Gradle 插件 -> 自定义 Transform -> 注册 Transform
-
-
- 参考：
  collapsed:: true
	- [刚学会Transform，你告诉我就要被移除了](https://juejin.cn/post/7114863832954044446)
-