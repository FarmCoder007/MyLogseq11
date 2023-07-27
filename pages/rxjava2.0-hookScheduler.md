title:: rxjava2.0-hookScheduler

- ```java
      public static Scheduler io() {
          return RxJavaPlugins.onIoScheduler(IO);
      }
  ```
- onIoScheduler
	- ```java
	      public static Scheduler onIoScheduler(@NonNull Scheduler defaultScheduler) {
	          Function<? super Scheduler, ? extends Scheduler> f = onIoHandler;
	          if (f == null) {
	              return defaultScheduler;
	          }
	          return apply(f, defaultScheduler);
	      }
	  ```
- 看上边得知，传入的为null 直接返回了，那么怎么设置这个onIoHandler
	- ```java
	      public static void setIoSchedulerHandler(@Nullable Function<? super Scheduler, ? extends Scheduler> handler) {
	          if (lockdown) {
	              throw new IllegalStateException("Plugins can't be changed anymore");
	          }
	          onIoHandler = handler;
	      }
	  
	  ```
- ## 应用
	- ```java
	          // TODO Hook  IO 传递进去的Hook
	          RxJavaPlugins.setIoSchedulerHandler(new Function<Scheduler, Scheduler>() {
	              @Override
	              public Scheduler apply(Scheduler scheduler) throws Exception {
	                  Log.d(L.TAG, "apply: 全局 监听 scheduler：" +scheduler);
	                  return scheduler;
	              }
	          });
	  
	          // TODO hook 给 IO 初始化
	          RxJavaPlugins.setInitIoSchedulerHandler(new Function<Callable<Scheduler>, Scheduler>() {
	              @Override
	              public Scheduler apply(Callable<Scheduler> schedulerCallable) throws Exception {
	                  Log.d(L.TAG, "apply: 全局 监听 init scheduler：" +schedulerCallable.call());
	                  return schedulerCallable.call();
	              }
	          });
	  ```