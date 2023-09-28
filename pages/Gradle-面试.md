# gradle构建流程/[[Gradle生命周期]]
	- 1、初始化阶段：执行settings.gradle脚本，确定哪些项目需要参与构建，单项目还是多项目，并且为这些项目创建一个Project实例
	- 2、配置阶段。解析各个project下的build.gradle文件，获取所有的task，形成有向无环图后执行依赖关系
	- 3、执行阶段，执行task
- # gradle怎么hook某个系统task->[原文](https://blog.csdn.net/weixin_33851604/article/details/91370909)
	- 1、在自定义插件的 `afterEvaluate`project初始化完成后回调 中寻找 某个task
	- 2、首先遍历 变体
		- 结合task完整名字去，mProject.tasks.findByPath 寻找具体task
		- 执行task.doFirst 执行hook操作
		- ```java
		  mProject.afterEvaluate {
		      processVariant()
		  }
		  void processVariant() throws NotFoundException {
		      // variant 一般有 debug 和 release
		      mProject.android.libraryVariants.all { variant ->
		          process(variant)
		      }
		  }
		  void process(variant){
		      String taskPath = 'bundle' + mVariant.name.capitalize()
		      Task bundleTask = mProject.tasks.findByPath(taskPath)
		      if (bundleTask == null) {
		          throw new RuntimeException("Can not find task ${taskPath}!")
		      }
		      bundleTask.doFirst {
		          // do hook
		      }    
		  }
		  ```
	- # [参考](https://blog.csdn.net/c6E5UlI1N/article/details/130838026)
- # Gradle插件和Gradle的区别
	- ==**gradle 插件**==负责 Android 个性化的打包工具，比如 aapt, dexbuilder, apkbuilder, jarsigner 等。
	- [[#red]]==**gradle**== 则是负责整个工程化大的流程