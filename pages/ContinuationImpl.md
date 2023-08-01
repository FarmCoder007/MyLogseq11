- # 1、源码
	- ```kotlin
	  
	  // State machines for named suspend functions extend from this class
	  internal abstract class ContinuationImpl(
	      completion: Continuation<Any?>?,
	      private val _context: CoroutineContext?
	  ) : BaseContinuationImpl(completion) {
	      constructor(completion: Continuation<Any?>?) : this(completion, completion?.context)
	  
	      public override val context: CoroutineContext
	          get() = _context!!
	  
	      @Transient
	      private var intercepted: Continuation<Any?>? = null
	  
	      public fun intercepted(): Continuation<Any?> =
	          intercepted
	              ?: (context[ContinuationInterceptor]?.interceptContinuation(this) ?: this)
	                  .also { intercepted = it }
	  
	      protected override fun releaseIntercepted() {
	          val intercepted = intercepted
	          if (intercepted != null && intercepted !== this) {
	              context[ContinuationInterceptor]!!.releaseInterceptedContinuation(intercepted)
	          }
	          this.intercepted = CompletedContinuation // just in case
	      }
	  }
	  
	  ```
- 首先这有个注释‘State machines for named restricted suspend functions extend from this class’，状态机为命名的受限的挂起函数从这个类继承。
- 这里学到了一个新概念“State machines”，那么什么是状态机呢？我们可以狭义的暂时理解就是Coroutine这个类。我们也可以延伸下就是就是控制整个协程挂起恢复的逻辑块，包括我们后边提到的switch代码块。
- 在这里我们还看到intercepted()和releaseIntercepted()这两个关于协程拦截器的方法。对没错kotlin协程也有拦截器。但是本文就不对这个做具体分析了有兴趣的同学可以自己研究。