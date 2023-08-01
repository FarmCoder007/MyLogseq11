- onCompletion
	- ```kotlin
	  public fun <T> Flow<T>.onCompletion(
	      action: suspend FlowCollector<T>.(cause: Throwable?) -> Unit
	  ): Flow<T> = unsafeFlow { 
	      try {
	          collect(this)
	      } catch (e: Throwable) {
	          ThrowingCollector(e).invokeSafely(action, e)
	          throw e
	      }
	      // Normal completion
	      val sc = SafeCollector(this, currentCoroutineContext())
	      try {
	          sc.action(null)
	      } finally {
	          sc.releaseIntercepted()
	      }
	  }
	  ```
- 该方法会在`flow`正常完成、被取消后执行，并且运行给定的`action`。
- 这里我们需要注意一点，就是`action`的函数类型是`(Throwable?) -> Unit`，即当程序发生异常时，或者调用`cancel()`会产生一个`CancellationException`时，这个异常信息会被传递到`onCompletion`的`action`中。
- 和下面要说的`catch`不同，这个操作符会抛出上游和下游的异常，和位置关系是不固定的，如果`Flow`在执行过程中没有异常，则`cause`会是`null`。
- 还有一个神奇操作，就是`action`的接收者类型是`FlowCollector`，所以在`action`中依旧可以调用`emit`发送数据，由上面源码可知，在`onCompletion`中`emit`的数据，会在最后被收集。