- # 介绍
	- 协程作用域，我们前面文章说了`launch`和`async`都是协程作用域`CoroutineScope`的扩展函数，我们来看一下这个`CoroutineScope`:
	- ```kotlin
	  public interface CoroutineScope {
	      /**
	       * The context of this scope.
	       * Context is encapsulated by the scope and used for implementation of coroutine builders that are extensions on the scope.
	       * Accessing this property in general code is not recommended for any purposes except accessing the [Job] instance for advanced usages.
	       *
	       * By convention, should contain an instance of a [job][Job] to enforce structured concurrency.
	       */
	      public val coroutineContext: CoroutineContext
	  }
	  ```
	- 会发现它就是一个简单的接口，而这个接口的唯一成员就是`CoroutineContext`，所以`CoroutineScope`只是对`CoroutineContext`**做了一层简单封装而已**，其核心能力还是`CoroutineContext`
- # 作用
	- 协程作用域的最大作用就是可以**方便批量控制协程**，说道批量控制协程，不由得想起来上篇文章所说的结构化并发，看下面代码：
		- ```kotlin
		  fun main() = runBlocking {
		      // 仅用于测试，生成环境不要使用这么简易的CoroutineScope
		      val scope = CoroutineScope(Job())
		  
		      scope.launch {
		          logX("First start!")
		          delay(1000L)
		          logX("First end!") // 不会执行
		      }
		  
		      scope.launch {
		          logX("Second start!")
		          delay(1000L)
		          logX("Second end!") // 不会执行
		      }
		  
		      scope.launch {
		          logX("Third start!")
		          delay(1000L)
		          logX("Third end!") // 不会执行
		      }
		  
		      delay(500L)
		      scope.cancel()
		      delay(1000L)
		  }
		  ```
	- 可以发现这里创建了一个`scope`，然后用这个`scope`启动了3个协程，当协程没有执行完成时，通过调用`scope`的`cancel`方法便可以取消这3个协程，上述代码打印如下:
	  collapsed:: true
		- ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d153aacabbdd4740ab24b4472874761b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)
	- 这同样体现了结构化并发的理念
- # 参考
	- [详细见](https://juejin.cn/post/7091850594474229796)