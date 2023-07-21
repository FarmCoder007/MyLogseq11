- 前面我们提过，分发器就是来调配请求任务的，内部会包含一个线程池。当异步请求时，会将请求任务交给线程池来执行。那分发器中默认的线程池是如何定义的呢？为什么要这么定义？
	- ```java
	      public synchronized ExecutorService executorService() {
	          if (executorService == null) {
	              executorService = new ThreadPoolExecutor(
	                      0, //核心线程
	                      Integer.MAX_VALUE, //最大线程
	                      60, //空闲线程闲置时间
	                      TimeUnit.SECONDS, //闲置时间单位
	                      new SynchronousQueue<Runnable>(), //线程等待队列
	                      Util.threadFactory("OkHttp Dispatcher", false) //线程创建工厂
	              );
	          }
	          return executorService;
	      }
	  }
	  ```
- 在OkHttp的分发器中的线程池定义如上，其实就和 Executors.newCachedThreadPool() 创建的线程一样。
- 首先==**核心线程为0**==，表示线程池不会一直为我们缓存线程，[[#red]]==**线程池中所有线程都是在60s内没有工作就会被回收。**==
- 而[[#red]]==**最大线程 Integer.MAX_VALUE 与等待队列 SynchronousQueue**== 的组合能够得到最大的吞吐量。即当需要线程池执行任务时，如果不存在空闲线程不需要等待，马上新建线程执行任务！等待队列的不同指定了线程池的不同排队机制。一般来说，等待队列 BlockingQueue 有： ArrayBlockingQueue 、 LinkedBlockingQueue 与 SynchronousQueue 。
-
- ## 假设向线程池提交任务时，核心线程都被占用的情况下：[[线程池等待队列]]
-
- 但是需要注意的时，我们都知道，进程的内存是存在限制的，而每一个线程都需要分配一定的内存。所以线程并不能无限个数。[[#red]]==**那么当设置最大线程数为 Integer.MAX_VALUE 时，OkHttp同时还有最大请求任务执行个数: 64的限制。这样即解决了这个问题同时也能获得最大吞吐**==。