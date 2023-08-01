- 这个也是从名字就可以看出其作用，**即协程异常处理**，我们看一下使用，代码如下
	- ```kotlin
	  fun main() = runBlocking {
	      val scope = CoroutineScope(Job() + mySingleDispatcher)
	      val myExceptionHandler = CoroutineExceptionHandler{_,thorwable ->
	          println("Catch exception : $thorwable")
	      }
	      scope.launch(CoroutineName("MyCoroutine") + myExceptionHandler) {
	          logX(coroutineContext[CoroutineDispatcher] == mySingleDispatcher)
	          val s: String? = null
	          s!!.length
	          delay(1000L)
	          logX("MyCoroutine end!")
	      }
	      delay(500L)
	      scope.cancel()
	      delay(1000L)
	  }
	  ```
- 这里在`scope`启动协程时，传递了协程名`Context`和异常处理`Context`，通过加号进行连接，然后这个协程的异常就可以被这个异常处理器捕获，打印如下：
- ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f2dec35f580b408ab1e7c2a07c27b077~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)