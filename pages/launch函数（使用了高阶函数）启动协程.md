- ## 1、方法介绍
	- ```kotlin
	  public fun CoroutineScope.launch(
	      context: CoroutineContext = EmptyCoroutineContext, // 上下文
	      start: CoroutineStart = CoroutineStart.DEFAULT,  // 启动模式
	      block: suspend CoroutineScope.() -> Unit // 协程体
	  ): Job 
	  ```
	- **`launch`是一个扩展函数**，它的接收者类型是`CoroutineScope`，**这个表示协程的作用域**，而Kotlin的扩展方法就表明`launch`就相当于`CoroutineScope`的成员方法。
	- ### 1、[[CoroutineStart协程启动模式]]
	- ### 2、[[CoroutineContext上下文]]：[详细](https://juejin.cn/post/7091850594474229796)
	- ### 3、协程体：开启后的{}里就是
	- 返回值是`Job`，**它代表是协程的句柄**，并不能返回协程的执行结果，这也就说明了`launch`启动的协程不能返回结果
- ## 2、launch函数启动
	- ```kotlin
	  fun main() {
	      GlobalScope.launch {
	          println("Hello World! 1")
	          delay(1000L)
	          println("Hello World! 2")
	      }
	      println("Hello World! 3")
	      Thread.sleep(2000L)
	  }
	  ```
	- 这里通过`launch`和后面的`lambda`其实就启动了一个协程，而且这个协程已经在运行了，或者可以这样说，`lambda`代码块即是协程运行的代码块，也可以理解为[[#red]]==**这段`lambda`代码就协程。**==
	- 注意和线程概念一样，**协程是一段程序，是一个动态执行的代码范围**，比如上面`lambda`中的3行代码就是一个协程，在这个`lambda`中可以打印出协程的信息，[[#red]]==**当代码执行完后，协程也就结束了。**==
	- 从这个角度来看，就更好理解"**协程是运行在线程上的Task**"这句话了，因为这个协程是运行在主线程上的Task。
- ## 3、特点
	- `launch`的协程一旦任务完成了，即便有结果，**也没办法直接返回被调用方**。
- ## 4、适用场景
	- 所以`launch`适合的场景是业务去执行且不需要得到其返回值的情况。
-