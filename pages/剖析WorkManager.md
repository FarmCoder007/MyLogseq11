- # WorkManager是干啥的
  collapsed:: true
	- WorkManager是一个可靠异步任务框架。相比其他调度框架WorkManager有以下优势：
	- 强大的任务调度策略，允许给任务设置约束，在耗电量，网络的条件下约束任务执行，达到最优效果。
	  兼容性平台广，支持安卓多个版本的兼容。
	  安全可靠，保证任务一定会被执行，即使在app异常退出后，下次进入依旧会调度任务。
	  策略灵活，支持任务回退策略、支持任务失败重新调度。
	  注意：WorkManager不是一种新的工作线程，它的出现不是为了替代其它类型的工作线程。工作线程通常立即运行，并在执行完成后给到用户反馈。而WorkManager不是即时的，它不能保证任务能立即得到执行。
- # 你将学到
	- WorkManager是如何初始化的？初始化了哪些东西？
	  WorkManager是如何做到即使APP异常退出后，调度任务仍可以在下次启动后重新调度？
	  WorkManager的耗电量，网络这些约束是怎么执行的？
	  WorkManager的周期性任务和一次性任务都是怎么调度的。
- # 源码分析
	- ## 初始化
		- 我们先来一张结构图从整体角度来看下WorkManager的初始化
		  collapsed:: true
			- ![image.png](../assets/image_1684427556910_0.png)
		- WorkManager由ContentProvider进行初始化，所以无需使用者手动初始化。
		  整个初始化阶段会初始化线程池、数据库、调度器、处理器、约束控制器组件。
		- ### WorkManagerInitializer
		  collapsed:: true
			- 我们可以看到看到WorkManager并不需要开发者手动调用初始化代码，而是巧妙的利用了ContentProvider来自动触发初始化任务的执行。
			- 我们是不是也可以利用ContentProvider来对我们App环境做一些初始化操作呢？比如说BuildConfig里的全局静态变量赋值用ContentProvider进行统一维护加载。
			- WorkManagerInitializer实际会调用WorkManagerImpl类完成具体的初始化任务，待会我们具体会讲到。
		- ### Configuration
			-