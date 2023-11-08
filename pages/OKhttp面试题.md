# 1、同步请求和异步请求的区别？
collapsed:: true
	- ## 同步请求
		- 同步会阻塞当前线程，
		- 场景，一次请求的响应结果是下次请求的参数。自己开线程去处理
	- ## 异步请求
		- 不阻塞当前线程，异步执行结果需要回调回来
- # 2、RealCall基础相关
  collapsed:: true
	- ## 1、每次请求都会new一个realCall
	- ## 2、一个realCall对象只能执行一次，会有判断
		- ```java
		    @Override public void enqueue(Callback responseCallback) {
		      // 一个RealCall只能执行一次
		      synchronized (this) {
		        if (executed) throw new IllegalStateException("Already Executed");
		        executed = true;
		      }
		      transmitter.callStart();
		      client.dispatcher().enqueue(new AsyncCall(responseCallback));
		    }
		  ```
- # 3、分发器Dispatcher
	- ## 1、分发器dispatch干嘛的？
		- 内部维护队列与线程池，完成请求调配
	- ## 2、调度器/分发器-异步请求流程
		- 1、发起一个异步请求[[#red]]==**realCall.enqueue**==（request）会调用到[[#red]]==**Dispatcher.enqueue()**==
		- 2、Dispatcher.enqueue() 把异步请求任务 AsyncCall 首先放入等待队列 readyAsyncCalls
		- 3、然后遍历 readyAsyncCalls等待队列，如果满足[[#red]]==**(执行队列里的任务数<64,相同Host 的任务数<5**==)两个条件，
			- 将异步请求任务，移入执行队列runningAsyncCalls，然后交给线程池去执行。
		- 4、runningAsyncCalls 里的任务执行完会调用finish（）方法，从runningAsyncCalls里移除，再遍历ready等待队列，找到满足[[#red]]==**(执行队列里的任务数<64,相同Host 的任务数<5**==)两个条件，放入执行队列running队列里执行下一个任务。如此往复
	- ## 3、调度器执行异步任务判断，相同host不能超过5个为什么？
		- 1、它可能参考 谷歌等浏览器是设置
		- 2、为了防止访问 服务器的过渡开销吧
	- ## 3、分发器一共使用了多少个队列:3个
		- ####  同步：runningSyncCalls
		- 异步：runningAsyncCalls   readyAsyncCalls
	- ## 4、同步suncRunning队列 和 异步的 ready 和 Asyncrunning队列 为啥用双向队列？
		- 双向队列  头尾 都可以插入删除。为了扩展吧，之前看源码没有这部分操作
	- ## 5、异步请求中，怎么把请求放入Running队列？
		- 遍历ready队列，当[[#green]]==**正在请求数量小于64 && 同一主机请求小于5 个**== 时 可以放入
	- ## 6、任务执行完，ready中的任务怎么添加到Running中的
		- 任务执行完，会调用finished 把 Running队列的这个执行完的任务移除掉，再次遍历ready队列，满足上述两个条件就会加入Running队列
	- ## 7、分发器线程池的工作类型
		- 无等待 最大并发。核心线程0 普通线程最大 无容量等待队列。全靠Runing队列64个限制
	- ## 8、 [[OkhttpClient可设置单例，也可不使用单例，带来的线程池问题？]]
	  collapsed:: true
		- ## 背景
			- Dipatcher调度器也是在okhttpClient中创建的，会导致[[#red]]==**每创建一个client就创建一个分发器，每个分发器创建一个线程池**==。
			- 所以创建多个 OkHttpClient 对象，就会对应数量的分发器，对应数量的线程池。每个请求再自己线程池里执行，线程池也失去了意义。
		- ## 问题解决
			- ## Retrofit结合RXJava时
				- 同步请求是在RXjava的io线程执行的
				- 异步请求，rxjava io线程里，最终还是okhttp分发器里线程池执行的。创建了多次线程
			- ## 58的做法：
				- 结合RXjava，在rxjava io线程中执行OKhttp的 同步任务。这样就不会使用OKhttp的线程池了。
				- 但是注意rxjava io线程 没有做数量限制。也避免同时大量请求
- # 4、[[拦截器面试题]]
- # 5、OKhttp中一次异步请求是怎样的？
	- 1、okhttpClient.newCall(request)  返回的是一个 RealCall 对象
	- 2、然后执行realCall.enqueue  实际里边调用的是  dispatcher .enqueue  将网络请求放到后台线程执行
	- 3、在后台线程  通过  getResponseWithInterceptorChain()  获取返回的响应response 放在callback 里回调回来
	- ## 参考整体流程
	  collapsed:: true
		- ![image.png](../assets/image_1689851761928_0.png){:height 877, :width 555}
- # 6、[[okhttp里的源码怎么处理Https证书的]]