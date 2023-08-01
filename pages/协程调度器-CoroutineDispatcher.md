- # Dispatchers源码
	- ```kotlin
	  public actual object Dispatchers {
	      @JvmStatic
	      public actual val Default: CoroutineDispatcher = DefaultScheduler
	  
	      @JvmStatic
	      public actual val Main: MainCoroutineDispatcher get() = MainDispatcherLoader.dispatcher
	  
	      @JvmStatic
	      public actual val Unconfined: CoroutineDispatcher = kotlinx.coroutines.Unconfined
	  
	      @JvmStatic
	      public val IO: CoroutineDispatcher = DefaultIoScheduler
	  
	      @DelicateCoroutinesApi
	      public fun shutdown() {
	          DefaultExecutor.shutdown()
	          // Also shuts down Dispatchers.IO
	          DefaultScheduler.shutdown()
	      }
	  }
	  ```
- # 类型CoroutineDispathcer
	- ```kotlin
	  public abstract class CoroutineDispatcher :
	      AbstractCoroutineContextElement(ContinuationInterceptor), ContinuationInterceptor
	  
	  public interface ContinuationInterceptor : CoroutineContext.Element
	  ```
	- 根据这个`CoroutineDispatcher`的继承关系，最终发现还是继承至`CoroutineContext`
- ## 1、作用
	- CoroutineDispatcher是用于协程调度的接口，它定义了协程运行时的上下文
- ## 2、[[CoroutineDispatcher协程调度器区别]]
- ## 3、例子
  collapsed:: true
	- 如果您需要在不同的线程池中运行协程，可以使用withContext()函数来指定不同的调度器。例如：
		- ```kotlin
		  import kotlinx.coroutines.*
		  fun main() {
		      GlobalScope.launch(Dispatchers.IO) {
		          // 使用IO调度器执行协程
		          val result = withContext(Dispatchers.Default) {
		              // 在默认调度器中执行协程
		              1 + 2
		          }
		          println("Result: $result")
		      }
		      runBlocking {
		          delay(1000L)
		      }
		  }
		  ```
		- 在这个例子中，我们使用launch()函数创建了一个协程，并指定了Dispatchers.IO调度器。在协程中，我们使用withContext()函数指定了不同的调度器，即Dispatchers.Default调度器。在withContext()函数中，我们执行了一个简单的加法操作，并返回结果。然后，在协程中输出了结果。注意，在withContext()函数中执行的协程也可以使用suspend关键字来定义。最后，我们使用runBlocking函数等待协程完成。
		  使用不同的调度器可以提高协程的性能和效率，因为它可以根据不同类型的工作来选择不同的线程池和执行策略。
- ## 4、自定义disPatcher
  collapsed:: true
	- ```kotlin
	  val mySingleDispatcher = Executors.newSingleThreadExecutor {
	      Thread(it,"MySingleThread").apply { isDaemon = true }
	  }.asCoroutineDispatcher()
	  ```
	- 我们创建了一个单线程的线程池，然后通过`asCoroutineDispathcer`方法转换为`Dispatcher`，然后我们来进行使用如下:
	- ```kotlin
	  fun main() = runBlocking(mySingleDispatcher) {
	      val user = getUserInfo()
	      logX(user)
	  }
	  
	  suspend fun getUserInfo(): String {
	      logX("Before IO Context.")
	      withContext(Dispatchers.IO) {
	          logX("In IO Context.")
	          delay(1000L)
	      }
	      logX("After IO Context.")
	      return "BoyCoder"
	  }
	  ```
	- 这里的`runBlocking`就传入我们自定义的`Dispather`，然后运行结果如下：
	  collapsed:: true
		- ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce5bafb00d234da481b40a6b9a58ae09~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)
	- 由于只有1个线程的线程池，所以切换IO线程时，会复用`Default`中的线程池。
	- 这其实也就印证了**协程是运行在线程上的Task这种说法**。