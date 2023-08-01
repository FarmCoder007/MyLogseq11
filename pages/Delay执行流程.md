- ## 源码
	- ```kotlin
	  public suspend fun delay(timeMillis: Long) {
	      if (timeMillis <= 0) return // don't delay
	      return suspendCancellableCoroutine sc@ { cont: CancellableContinuation<Unit> ->
	          // if timeMillis == Long.MAX_VALUE then just wait forever like awaitCancellation, don't schedule.
	          if (timeMillis < Long.MAX_VALUE) {
	              // 延时后 恢复续体
	              cont.context.delay.scheduleResumeAfterDelay(timeMillis, cont)
	          }
	      }
	  }
	  ```
- cont为传入的Continuation
- ## 1、cont.context.delay
	- ```kotlin
	  internal val CoroutineContext.delay: Delay get() = get(ContinuationInterceptor) as? Delay ?: DefaultDelay
	  ```
	- 如果`get(ContinuationInterceptor)`是`Delay`类型对象，那么直接返回该对象，如果不是返回`DefaultDelay`变量，看一下`DefaultDelay`初始化可以知道，它是一个`DefaultExecutor`对象，继承了`EventLoopImplBase`类。
- ## 2、看EventLoopImplBase的方法
	- ```kotlin
	      override fun scheduleResumeAfterDelay(timeMillis: Long, continuation: CancellableContinuation<Unit>) {
	          val timeNanos = delayToNanos(timeMillis)
	          if (timeNanos < MAX_DELAY_NS) {
	              val now = nanoTime()
	              DelayedResumeTask(now + timeNanos, continuation).also { task ->
	                  /*
	                   * Order is important here: first we schedule the heap and only then
	                   * publish it to continuation. Otherwise, `DelayedResumeTask` would
	                   * have to know how to be disposed of even when it wasn't scheduled yet.
	                   */
	                  schedule(now, task)
	                  continuation.disposeOnCancellation(task)
	              }
	          }
	      }
	  ```
- # # [深究Kotlin协程delay函数源码实现](https://www.jianshu.com/p/1d9dcd331b9c)