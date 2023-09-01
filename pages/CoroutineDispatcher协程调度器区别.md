## Dispatchers.Default：
	- 默认调度器，用于**CPU密集型任务的线程池**，一般来说它内部的线程个数与机器CPU核心数量保持一致。**共享线程池**
- ## Dispatchers.IO：
	- 用于**IO密集型任务的线程池**，内部线程数量较多，一般为64个。
	- 适用于执行I/O密集型的工作，使用专门的线程池执行协程。
- ## Dispatchers.Main：
	- 用于Android平台上的主线程，用于更新UI或执行短时间的计算操作。
	- Main
		- ```kotlin
		  internal object MainDispatcherLoader {
		      @JvmField
		      val dispatcher: MainCoroutineDispatcher = loadMainDispatcher()
		  
		      private fun loadMainDispatcher(): MainCoroutineDispatcher {
		          return try {
		              val factories = MainDispatcherFactory::class.java.let { clz ->
		                  ServiceLoader.load(clz, clz.classLoader).toList()
		              }
		              factories.maxBy { it.loadPriority }?.tryCreateDispatcher(factories)
		                  ?: MissingMainCoroutineDispatcher(null)
		          } catch (e: Throwable) {
		              MissingMainCoroutineDispatcher(e)
		          }
		      }
		  }
		  ```
		- 在 Android 当中，协程框架通过注册 `AndroidDispatcherFactory` 使得 `Main` 最终被赋值为 `HandlerDispatcher` 的实例，有兴趣的可以去看下  kotlinx-coroutines-android 的源码实现。
- ## Dispatchers.Unconfined：
	- 没有指定线程池的调度器，运行在当前线程或继承了调用者线程的线程池中。
	- 在Kotlin中，协程默认会运行在Dispatchers.Default调度器中，这意味着它们会使用共享的线程池。