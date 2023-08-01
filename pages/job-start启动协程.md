- 在前面我们所说的创建协程方法，都是在创建完协程后立即执行了，想使用`start()`方法就需要和懒加载配合使用：
- ```kotlin
  fun main() = runBlocking {
      //                  变化在这里
      //                      ↓
      val job = launch(start = CoroutineStart.LAZY) {
          logX("Coroutine start!")
          delay(1000L)
      }
      delay(500L)
      job.log()
      job.start()     // 变化在这里
      job.log()
      delay(500L)
      job.cancel()
      delay(500L)
      job.log()
      delay(2000L)
      logX("Process end!")
  }
  ```
- 上面代码我们使用懒加载模式来启动协程，虽然已经`delay` 500ms了，我们可以因为没有调用`start()`，这个`job`其实是没有启动的，我们来看一下打印：
- ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3ff997da4aa49fcb5ed8f4164ba03c6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)
- 第一个红框内就是在调用`start()`前的状态，是不活跃的，当调用`start()`后，协程 2开始运行，但是运行时间是1000ms，当协程 1中再调用`cancel()`后，协程 2就变成了取消状态。