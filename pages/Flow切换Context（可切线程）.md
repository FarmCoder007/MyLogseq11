- 前面说了`Flow`在复杂业务上可以取代`RxJava`，而复杂的业务经常需要频繁地切换工作的线程，对于耗时任务，我们需要在线程池中执行，对于UI任务，我们需要在主线程上执行，而在`Flo`w当中，我们可以借助`flowOn`这个中间操作符便可以完成需求。
- ## flowOn 切换上游的上下文
  collapsed:: true
	- ```kotlin
	  fun main() = runBlocking {
	      flow {
	          logX("Start")
	          emit(1)
	          logX("Emit 1")
	          emit(2)
	          logX("Emit 2")
	          emit(3)
	          logX("Emit 3")
	      }.filter {
	          logX("Filter: $it")
	          it > 2
	      }.flowOn(Dispatchers.IO)
	          .collect{
	              logX("Collect $it")
	          }
	  }
	  ```
	- 和前面的`catch`操作符的作用域一样，它的作用域**只对它的上游有作用**，上面代码的运行结果如下：
	  id:: 64c5ddbe-5a76-4ea6-bcb5-5eaa5b7418ab
	- ```kotlin
	  ================================
	  Start
	  Thread:DefaultDispatcher-worker-1 @coroutine#2
	  ================================
	  ================================
	  Filter: 1
	  Thread:DefaultDispatcher-worker-1 @coroutine#2
	  ================================
	  ================================
	  Emit 1
	  Thread:DefaultDispatcher-worker-1 @coroutine#2
	  ================================
	  ================================
	  Filter: 2
	  Thread:DefaultDispatcher-worker-1 @coroutine#2
	  ================================
	  ================================
	  Emit 2
	  Thread:DefaultDispatcher-worker-1 @coroutine#2
	  ================================
	  ================================
	  Filter: 3
	  Thread:DefaultDispatcher-worker-1 @coroutine#2
	  ================================
	  ================================
	  Emit 3
	  Thread:DefaultDispatcher-worker-1 @coroutine#2
	  ================================
	  ================================
	  Collect 3
	  Thread:main @coroutine#1
	  ================================
	  
	  Process finished with exit code 0
	  
	  ```
	- 可以发现在`Flow`中，`emit`和`filter`所在的协程2运行在子线程中，而`collect`运行在主线程中，这也就印证了前面所说的作用域问题。
- ## launchIn 指定 CoroutineScope
	- 前面和`catch`操作符一样的问题，它的作用域范围只是它前面的部分，同时假如多次调用`flowOn`则当前`flowOn`的范围是到上一个`flowOn`，这个也非常好理解。
	- 那下面就来解决如何指定下游操作符的运行线程，或者更直接的就是`flowOn`后面的中间操作符和终止操作符所运行的线程，Kotlin这里提供了一个叫做`launchIn`的操作符，它可以把一部分操作指定`Scope`，示例代码如下：
		- ```kotlin
		  //新建一个Dispatcher
		  val mySingleDispatcher = Executors.newSingleThreadExecutor {
		      Thread(it, "MySingleThread").apply {
		          isDaemon = true
		      }
		  }.asCoroutineDispatcher()
		  
		  fun main() = runBlocking {
		      //创建一个CoroutineScope
		      val scope = CoroutineScope(mySingleDispatcher)
		      flow {
		          logX("Start")
		          emit(1)
		          logX("Emit 1")
		          emit(2)
		          logX("Emit 2")
		          emit(3)
		          logX("Emit 3")
		      }.flowOn(Dispatchers.IO)
		          .filter {
		          logX("Filter: $it")
		          it > 2
		      }.onEach {
		              logX("onEach $it")
		          }
		          //指定运行在哪个协程范围内
		          .launchIn(scope)
		  
		      delay(1000)
		  }
		  ```
		- 这里的代码比较特殊，我们可以肯定地是发射数据地代码肯定是在IO线程中，而`filter`和`onEach`中的代码运行在什么地方呢？
		- 通过运行我们会发现这个俩部分代码会运行在我们自定义的线程池中：
		  collapsed:: true
			- ```kotlin
			  ================================
			  Start
			  Thread:DefaultDispatcher-worker-1 @coroutine#3
			  ================================
			  ================================
			  Emit 1
			  Thread:DefaultDispatcher-worker-1 @coroutine#3
			  ================================
			  ================================
			  Emit 2
			  Thread:DefaultDispatcher-worker-1 @coroutine#3
			  ================================
			  ================================
			  Emit 3
			  Thread:DefaultDispatcher-worker-1 @coroutine#3
			  ================================
			  ================================
			  Filter: 1
			  Thread:MySingleThread @coroutine#2
			  ================================
			  ================================
			  Filter: 2
			  Thread:MySingleThread @coroutine#2
			  ================================
			  ================================
			  Filter: 3
			  Thread:MySingleThread @coroutine#2
			  ================================
			  ================================
			  onEach 3
			  Thread:MySingleThread @coroutine#2
			  ================================
			  
			  Process finished with exit code 0
			  ```
	- ## [[onEach操作符]]
	- ## [[launchIn]]