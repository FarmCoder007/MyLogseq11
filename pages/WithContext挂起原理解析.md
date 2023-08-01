- ## 进入withContext源码
	- ```kotlin
	  public suspend fun <T> withContext(
	      context: CoroutineContext,
	      block: suspend CoroutineScope.() -> T
	  ): T {
	      contract {
	          callsInPlace(block, InvocationKind.EXACTLY_ONCE)
	      }
	      return suspendCoroutineUninterceptedOrReturn sc@ { uCont ->
	          // compute new context
	          val oldContext = uCont.context
	          // Copy CopyableThreadContextElement if necessary
	          val newContext = oldContext.newCoroutineContext(context)
	          // always check for cancellation of new context
	          newContext.ensureActive()
	          // FAST PATH #1 -- new context is the same as the old one
	          if (newContext === oldContext) {
	              val coroutine = ScopeCoroutine(newContext, uCont)
	              return@sc coroutine.startUndispatchedOrReturn(coroutine, block)
	          }
	          // FAST PATH #2 -- the new dispatcher is the same as the old one (something else changed)
	          // `equals` is used by design (see equals implementation is wrapper context like ExecutorCoroutineDispatcher)
	          if (newContext[ContinuationInterceptor] == oldContext[ContinuationInterceptor]) {
	              val coroutine = UndispatchedCoroutine(newContext, uCont)
	              // There are changes in the context, so this thread needs to be updated
	              withCoroutineContext(coroutine.context, null) {
	                  return@sc coroutine.startUndispatchedOrReturn(coroutine, block)
	              }
	          }
	          // SLOW PATH -- use new dispatcher
	          val coroutine = DispatchedCoroutine(newContext, uCont)
	          //开启协程
	          block.startCoroutineCancellable(coroutine, coroutine)
	          //获取协程结果
	          coroutine.getResult()
	      }
	  }
	  ```
- ## 返回suspendCoroutineUninterceptedOrReturn这个函数值
- suspendCoroutineUninterceptedOrReturn 传入的uCount 实参即为父协程的协程体
- 看getResult->COROUTINE_SUSPENDED
	- ```kotlin
	  #DispatchedCoroutine类里
	  fun getResult(): Any? {
	      //先判断是否需要挂起
	      if (trySuspend()) return COROUTINE_SUSPENDED
	      //如果无需挂起，说明协程已经执行完毕
	      //那么需要将返回值返回
	      val state = this.state.unboxState()
	      if (state is CompletedExceptionally) throw state.cause
	      //强转返回值到对应的类型
	      return state as T
	  }
	  
	  
	  。
	  ```
- ### 看trySuspend（）
	- ```kotlin
	  private fun trySuspend(): Boolean {
	      //_decision 原子变量，三种值可选
	      //private const val UNDECIDED = 0 默认值，未确定是1还是2
	      //private const val SUSPENDED = 1 挂起
	      //private const val RESUMED = 2   恢复
	      _decision.loop { decision ->
	          when (decision) {
	              //若当前值为默认值，则修改为挂起，并且返回ture，表示需要挂起
	              UNDECIDED -> if (this._decision.compareAndSet(UNDECIDED, SUSPENDED)) return true
	              //当前已经是恢复状态，无需挂起，返回false
	              RESUMED -> return false
	              else -> error("Already suspended")
	          }
	      }
	  }
	  ```
	- 返回true
- 这里 getResult返回COROUTINE_SUSPENDED,实际是withCOntext返回 COROUTINE_SUSPENDED