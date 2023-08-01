- - `launchIn`操作又是啥呢 为什么没有终止操作符，这里的代码依旧可以运行，我们来看一下源码：
- ```kotlin
  public fun <T> Flow<T>.launchIn(scope: CoroutineScope): Job = scope.launch {
      collect() // tail-call
  }
  ```
- 会发现`launchIn`是`Flow`的扩展函数，而且直接在`scope`调用`launch`启动了一个新协程，而协程中调用了`collect()`，这也就说明了为什么这里没有终止操作符的原因。
- 所以严格意义上说`launchIn`算是一个终止操作符，把它上游的代码都分发到指定的线程当中。
- 同时这个launchIn操作符在源码中有特殊的使用说明，代码注释如下：
	- ```kotlin
	  This operator is usually used with onEach, 
	  onCompletion and catch operators to process all emitted values handle an exception that might occur in the upstream 
	  flow or during processing, 
	  for example:
	  flow
	      .onEach { value -> updateUi(value) }
	      .onCompletion { cause -> updateUi(if (cause == null) "Done" else "Failed") }
	      .catch { cause -> LOG.error("Exception: $cause") }
	      .launchIn(uiScope)
	  
	  ```
- 即这个操作符经常和`onEach`、`onCompletion`、`catch`一起使用，处理所有发出的值，处理可能在上游或者处理过程中发生的异常。