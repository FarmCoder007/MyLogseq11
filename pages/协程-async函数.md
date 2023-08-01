- ## 源码,使用了高阶函数
	- ```kotlin
	  public fun <T> CoroutineScope.async(
	      context: CoroutineContext = EmptyCoroutineContext,
	      start: CoroutineStart = CoroutineStart.DEFAULT,
	      block: suspend CoroutineScope.() -> T
	  ): Deferred<T> {
	      val newContext = newCoroutineContext(context)
	      val coroutine = if (start.isLazy)
	          LazyDeferredCoroutine(newContext, block) else
	          DeferredCoroutine<T>(newContext, active = true)
	      coroutine.start(start, coroutine, block)
	      return coroutine
	  }
	  ```
- ## 作用
	- 1、进行线程切换
	- 2、根据高阶函数执行结果，进行类型推断拿到 job(子类Deferred)，进而获取异步执行结果