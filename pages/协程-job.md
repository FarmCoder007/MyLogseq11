- # 源码
	- 前面说了`CoroutineScope`还是封装的`CoroutineContext`的话，那Job就是一个真正的`CoroutineContext`了，我们来看一下源码：
	- ```kotlin
	  public interface Job : CoroutineContext.Element {}
	  ```
	- ```kotlin
	  public interface Element : CoroutineContext {}
	  ```
	- 可以发现这里通过2层继承，Job就是CoroutineContext的子类。
- ## 作用
	- 在Kotlin协程中，Job是一个用于管理协程的接口，它可以用来启动、取消、等待和检查协程的状态。
	- -使用`Job`**监测协程的生命周期状态**；
	- -使用`Job`**操控协程**。
- ## job接口有两个重要的子类：
	- JobSupport
		- JobSupport是用于启动、取消和检查协程状态的基本实现
	- Deferred
		- 而Deferred则继承了JobSupport，并添加了一个可以返回结果的[[#red]]==**await()函数**==
	- 通常我们使用launch()函数创建一个Job实例，这个Job实例代表着我们创建的协程。在协程启动后，我们可以使用这个Job实例来管理协程的状态。
- ## Job接口的常用方法：
	- cancel()：取消协程，如果协程已经完成或被取消，则不起作用。
	- [join()](https://juejin.cn/post/7091842873938673671)：等待协程完成，如果协程已经完成或被取消，则立即返回。
	- isActive：检查协程是否处于活动状态，即是否已启动且未被取消。
	- isCompleted：检查协程是否已经完成，即已经执行完毕或被取消。
	- - `invokeOnCompletion`是一个回调，在协程执行完成时回调。
	- `start()`可以配合懒加载启动协程
- 使用Job接口，我们可以对协程进行管理，以避免在不需要协程时浪费系统资源。此外，Job接口还允许我们使用async()函数等待协程返回结果，以便我们可以在协程完成后获取结果。
- ## 协程的外部管程状态
  collapsed:: true
	- 主要分为:**是否活跃**、**是否已经取消**和**是否已经完成**3种状态，我们通过下面打印来获取：
		- ```kotlin
		  fun Job.log() {
		      logX("""
		          isActive = $isActive
		          isCancelled = $isCancelled
		          isCompleted = $isCompleted
		      """.trimIndent())
		  }
		  
		  /**
		   * 控制台输出带协程信息的log
		   */
		  fun logX(any: Any?) {
		      println("""
		  ================================
		  $any
		  Thread:${Thread.currentThread().name}
		  ================================""".trimIndent())
		  }
		  
		  ```
	- 通过调用`Job`的`isActive`属性来判断`Job`是否处于活跃状态。
	- 通过调用`Job`的`isCancelled`属性来判断`Job`是否处于已经取消状态。
	- 通过调用`Job`的`isCompleted`属性来判断`Job`是否处于已完成状态。
- ## [[job-start启动协程]]
- ## job 内部状态
	- ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e7743d8628b64065b3d8248f26e01350~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)
-
- # [[Job面试题]]