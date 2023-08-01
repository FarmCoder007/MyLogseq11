- 源码
  collapsed:: true
	- ```kotlin
	  public fun <T> Flow<T>.onStart(
	      action: suspend FlowCollector<T>.() -> Unit
	  ): Flow<T> = unsafeFlow {
	      val safeCollector = SafeCollector<T>(this, currentCoroutineContext())
	      try {
	          safeCollector.action()
	      } finally {
	          safeCollector.releaseIntercepted()
	      }
	      collect(this) // directly delegate
	  }
	  ```
- 首先，调用`onStart`方法会返回一个新的`Flow`，其次`action`函数类型接收者是`FlowCollector`，所以可以在`onStart`的`lambda`中调用`emit`方法，并且会在上游发送数据前先执行，比如如下代码
- ```kotlin
  flowOf("a", "b", "c")
      .onStart { emit("Begin") }
      .collect { println(it) }
  ```
- 这里会打印`Begin,a,b,c`，也会说明`onStart`方法的执行优先级非常高，和位置无关。
- 这里衍生一点，就是`SharedFlow`，因为`Flow`是冷的，而`SharedFlow`是热的，所以当`onStart`和`SharedFlow`一起使用时，就无法保证`onStart`一定会在发送数据之前执行，这时可以使用`onSubscription`其他API来完成，关于这点，后面说`SharedFlow`再细说。