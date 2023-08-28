# gradle构建流程
	- 1、初始化阶段：执行settings.gradle脚本
	- 2、配置阶段。执行项目的build.gradle
	- 3、执行阶段，执行task
- # gradle的生命周期
- # gradle怎么hook某个系统task->[原文](https://blog.csdn.net/weixin_33851604/article/details/91370909)
	- 1、在自定义插件的 `afterEvaluate`project初始化完成后回调 中寻找 某个task
	- 2、首先遍历 变体
	  collapsed:: true
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