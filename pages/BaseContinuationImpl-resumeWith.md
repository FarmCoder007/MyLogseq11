- # 源码
  collapsed:: true
	- ```kotlin
	  internal abstract class BaseContinuationImpl(
	      // This is `public val` so that it is private on JVM and cannot be modified by untrusted code, yet
	      // it has a public getter (since even untrusted code is allowed to inspect its call stack).
	      public val completion: Continuation<Any?>?
	  ) : Continuation<Any?>, CoroutineStackFrame, Serializable {
	      // result为异步回调结果
	      // 此事实用于使用递归  恢复
	      public final override fun resumeWith(result: Result<Any?>) {
	          // 此循环在current.resumeWith（param）中展开递归，以在恢复时生成更清晰、更短的堆栈跟踪
	          var current = this// 注解1 
	          var param = result // 调用resumeWith方想要回调的结果
	          while (true) {
	           
	             // 可以精确跟踪已恢复挂起的调用堆栈的哪个部分
	              probeCoroutineResumed(current)
	              with(current) {
	                  val completion = completion!! // 注释2 
	                  val outcome: Result<Any?> =
	                      try {
	                          val outcome = invokeSuspend(param)  // 注释3 
	                          if (outcome === COROUTINE_SUSPENDED) return  // 注释4 
	                          Result.success(outcome)
	                      } catch (exception: Throwable) {
	                          Result.failure(exception)
	                      }
	                  releaseIntercepted() // this state machine instance is terminating
	                  if (completion is BaseContinuationImpl) {  // 注释5 
	                      // unrolling recursion via loop
	                      current = completion
	                      param = outcome
	                  } else {
	                      // top-level completion reached -- invoke and return
	                      // 达到顶层完成--调用并返回。
	                      // 非 BaseContinuationImpl，即顶层传入的AbstractCoroutine 调用resumeWith。完成整个恢复
	                      completion.resumeWith(outcome)  // 注释6 
	                      return
	                  }
	              }
	          }
	      }
	  
	      protected abstract fun invokeSuspend(result: Result<Any?>): Any?
	  
	  }
	  ```
	- ## 注释1：var current = this，
		- 即哪个挂起函数的`continuation实现类continuationImpl`调用的resumeWith，进行就是哪个Continuation，详细看完成流程里的
	- ## 注释2：val completion = completion
		- 取出当前continuationImpl里，包裹的上层的Continuation。递归传值用。可以实现子协程向上层 的continuationImpl的转换。达到结果传递
	- ## 注释3：invokeSuspend(param)
		- case1、如果是Launch函数里第一个Continuation调用的则是 状态机入口函数调用
		- case2、内部挂起函数的continuationImpl 调用的 invokeSuspend，
			- 要么执行到到挂起函数。return，挂起协程
			- 要么非挂起，恢复用，将子协程回传来的值回传给父协程
	- ## 注释5、completion is BaseContinuationImpl
		- 判断当前continuationImpl 里的completion  是否还是BaseContinuationImpl
		- 相当于取出父协程的continuationImpl ，执行下次递归。把值传递给父协程用
	- ## 注释6、恢复完成completion.resumeWith(outcome)
		- 最终执行到StandaloneCoroutine 的resumeWith
- # [[BaseContinuationImpl-resumeWith作用]]：
- # 1、BaseContinuationImpl源码看resumeWith流程
  title:: BaseContinuationImpl-resumeWith
  collapsed:: true
	- 我们接着回到BaseContinuationImpl这个类上来。
	- ## 1、while (true)死循环，查询协程已经进入了可resume状态
		- 这个死循环干啥的上边有个注释写的很清楚。‘This loop unrolls recursion in current.resumeWith(param) to make saner and shorter stack traces on resume’这个循环是为了确保协程已经进入了可resume状态。那啥时候resume？不知道我们接着往后看。
			- ```kotlin
			  val outcome: Result<Any?> =
			            try {
			              val outcome = invokeSuspend(param)
			              if (outcome === COROUTINE_SUSPENDED) return
			              Result.success(outcome)
			            } catch (exception: Throwable) {
			              Result.failure(exception)
			            }
			  ```
		- 这个就很明白了，调用invokeSuspend方法，如果是‘COROUTINE_SUSPENDED’[[#red]]==**挂起状态呢那就跳出这个方法**==等待下次被调用。
		- 如果当前[[#red]]==**不是挂起状态呢，就是可resume**==，那就把结果包装到Result.success()的参数里继续往后走。
	- ## 2、下边代码completion is BaseContinuationImp
		- 如果取出来的completion  （就是父协程的ContinuationImpl）还是BaseContinuationImpl，进行赋值等待下次递归while循环调用
		- 否则就是顶层最后一个‘completion’，那么就调用completion.resumeWith(outcome)返回。
	- 这里可以看到这个while(true)不仅仅是校验invokeSuspend结果是不是非挂起状态。他还负责跟递归一样一层一层的调用每个Continuation的invokeSuspend直到调用到了最顶层的。
-