- # 1、介绍
	- 从名字就可以看出这个是用来**给协程命名**的，比如下面代码
	- ```kotlin
	  fun main() = runBlocking {
	      val scope = CoroutineScope(Job() + mySingleDispatcher)
	      scope.launch(CoroutineName("MyCoroutine")) {
	          logX(coroutineContext[CoroutineDispatcher] == mySingleDispatcher)
	          delay(1000L)
	          logX("MyCoroutine end!")
	      }
	      delay(500L)
	      scope.cancel()
	      delay(1000L)
	  }
	  ```
	- 这里我们在启动协程时传入一个名字，那打印如下:
	- ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a7e3c1c71824b39a9b5567647eea399~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)
	- 会发现这里在线程名后@的协程名，就是我们命名的，而#2是一个自增的ID。