- # 一、简介
	- Dispatcher（分发器）分发流程,内部维护请求队列与线程池，负责请求任务的分发
	- 分发器就是来调配请求任务的，内部会包含一个线程池。可以在创建 OkHttpClient 时，传递我们自己定义的线程池来创建分发器。
- # 二、成员
  collapsed:: true
	- ```java
	  //异步请求同时存在的最大请求
	  private int maxRequests = 64;
	  //异步请求同一域名同时存在的最大请求
	  private int maxRequestsPerHost = 5;
	  //闲置任务(没有请求时可执行一些任务，由使用者设置)
	  private @Nullable Runnable idleCallback;
	  //异步请求使用的线程池
	  private @Nullable ExecutorService executorService;
	  //异步请求等待执行队列
	  private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();
	  //异步请求正在执行队列
	  private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();
	  //同步请求正在执行队列
	  private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();
	  ```
- # 三、同步请求
  collapsed:: true
	- ### realCall.execute
		- ```java
		    override fun execute(): Response {
		      check(executed.compareAndSet(false, true)) { "Already Executed" }
		  
		      timeout.enter()
		      callStart()
		      try {
		        client.dispatcher.executed(this)
		        return getResponseWithInterceptorChain()
		      } finally {
		        client.dispatcher.finished(this)
		      }
		    }
		  ```
	- ### client.dispatcher.executed(this)
		- ### 因为同步请求不需要线程池，也不存在任何限制。所以分发器用同步队列记录下。仅做一下记录。
		- ```java
		    @Synchronized internal fun executed(call: RealCall) {
		      runningSyncCalls.add(call)
		    }
		  ```
	- ### 直接调用getResponseWithInterceptorChain 获取响应
	- ### client.dispatcher.finished(this) 任务执行完 分发器结束该任务从runningSyncCalls同步队列中移除
- # 四、**异步请求**
  collapsed:: true
	- ## 流程图
	  collapsed:: true
		- ![image.png](../assets/image_1684428964414_0.png)
	- ### RealCall.kt  enqueue
	  collapsed:: true
		- ```kotlin
		  
		    override fun enqueue(responseCallback: Callback) {
		      // 一个RealCall只能执行一次
		      check(executed.compareAndSet(false, true)) { "Already Executed" }
		  
		      callStart()
		        // 交给分发器
		      client.dispatcher.enqueue(AsyncCall(responseCallback))
		    }
		  ```
	- ### dispatcher.enqueue
	  collapsed:: true
		- ```java
		    internal fun enqueue(call: AsyncCall) {
		      synchronized(this) {
		        // 1放入队列
		        readyAsyncCalls.add(call)
		  
		        // Mutate the AsyncCall so that it shares the AtomicInteger of an existing running call to
		        // the same host.
		        if (!call.call.forWebSocket) {
		          val existingCall = findExistingCallWithHost(call.host)
		          if (existingCall != null) call.reuseCallsPerHostFrom(existingCall)
		        }
		      }
		      // 执行
		      promoteAndExecute()
		    }
		  ```
		- 1、Dispatcher.enqueue() 把一个任务 [[#red]]==**AsyncCall (一个runnable )**==首先放入等待队列 readyAsyncCalls(双向队列)
		- 2、执行promoteAndExecute
	- ### dispatcher.promoteAndExecute
		- ```java
		   private fun promoteAndExecute(): Boolean {
		      this.assertThreadDoesntHoldLock()
		  
		      val executableCalls = mutableListOf<AsyncCall>()
		      val isRunning: Boolean
		      synchronized(this) {
		        val i = readyAsyncCalls.iterator()
		        while (i.hasNext()) {
		          val asyncCall = i.next()
		  
		          if (runningAsyncCalls.size >= this.maxRequests) break // Max capacity.
		          if (asyncCall.callsPerHost.get() >= this.maxRequestsPerHost) continue // Host max capacity.
		  
		          i.remove()
		          asyncCall.callsPerHost.incrementAndGet()
		          executableCalls.add(asyncCall)
		          runningAsyncCalls.add(asyncCall)
		        }
		        isRunning = runningCallsCount() > 0
		      }
		  
		      for (i in 0 until executableCalls.size) {
		        val asyncCall = executableCalls[i]
		        asyncCall.executeOn(executorService)
		      }
		  
		      return isRunning
		    }
		  ```
		- 条件1 当正在执行的任务未超过最大限制64，
		- 条件2 同时 runningCallsForHost(call) < maxRequestsPerHost 同一Host的请求不超过5个，
		- 则会添加到正在执行队列，同时提交给线程池见 asyncCall.executeOn
	- ### asyncCall.executeOn线程池处理
	  collapsed:: true
		- ```java
		      fun executeOn(executorService: ExecutorService) {
		        client.dispatcher.assertThreadDoesntHoldLock()
		   
		        var success = false
		        try {
		          // 核心的 线程调用 开始执行 把runnable扔进去  这时候 切到后台线程
		          executorService.execute(this)
		          success = true
		        } catch (e: RejectedExecutionException) {
		          val ioException = InterruptedIOException("executor rejected")
		          ioException.initCause(e)
		          noMoreExchanges(ioException)
		          responseCallback.onFailure(this@RealCall, ioException)
		        } finally {
		          if (!success) {
		            client.dispatcher.finished(this) // This call is no longer running!
		          }
		        }
		      }
		  ```
		- 执行executorService.execute(this) 线程池执行这个runnable。那么看asyncCall run方法就行了
	- ### asyncCall.run
	  collapsed:: true
		- ```java
		  override fun run() {
		        threadName("OkHttp ${redactedUrl()}") {
		          var signalledCallback = false
		          timeout.enter()
		          try {
		            // 重点1 拿到返回的响应 
		            val response = getResponseWithInterceptorChain()
		            signalledCallback = true
		            // 2 callBack 返回响应   
		            responseCallback.onResponse(this@RealCall, response)
		          } catch (e: IOException) {
		            if (signalledCallback) {
		              // Do not signal the callback twice!
		              Platform.get().log("Callback failure for ${toLoggableString()}", Platform.INFO, e)
		            } else {
		              responseCallback.onFailure(this@RealCall, e)
		            }
		          } catch (t: Throwable) {
		            cancel()
		            if (!signalledCallback) {
		              val canceledException = IOException("canceled due to $t")
		              canceledException.addSuppressed(t)
		              responseCallback.onFailure(this@RealCall, canceledException)
		            }
		            throw t
		          } finally {
		            // 重点2 告诉 调度器 这个AsyncCall结束了
		            client.dispatcher.finished(this)
		          }
		        }
		      }
		  ```
		- 1、getResponseWithInterceptorChain获取响应。见拦截器那节
		- 2、调度器结束这个AsyncCall(结束一个才能调度下一个任务)
	- ### dispatcher.finished
		- ```kotlin
		    internal fun finished(call: AsyncCall) {
		      call.callsPerHost.decrementAndGet()
		      finished(runningAsyncCalls, call)
		    }
		  ```
	- ### finished
	  collapsed:: true
		- ```kotlin
		    internal fun finished(call: RealCall) {
		      finished(runningSyncCalls, call)
		    }
		  
		    private fun <T> finished(calls: Deque<T>, call: T) {
		      val idleCallback: Runnable?
		      synchronized(this) {
		        if (!calls.remove(call)) throw AssertionError("Call wasn't in-flight!")
		        idleCallback = this.idleCallback
		      }
		  
		      val isRunning = promoteAndExecute()
		  
		      if (!isRunning && idleCallback != null) {
		        idleCallback.run()
		      }
		    }
		  ```
		- 1、calls.remove(call) 从runningSyncCalls 正在执行队列 移除这个任务
		- 2、再次调用  promoteAndExecute 轮训等待队列找可执行的任务
	- ## 总结：[[调度器异步请求流程]]
- # 六、[[一个请求的请求流程]]
- # 七、[[分发器Dispatcher的线程池]]
-