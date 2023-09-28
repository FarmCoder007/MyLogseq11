title:: Observable.interval取消原理

- 代码使用
	- ```java
	  Observable.interval(1,2, TimeUnit.SECONDS) // 返回 ObservableInterval，看他的订阅
	                  .subscribe(new Observer<Long>() {
	                      @Override
	                      public void onSubscribe(@NonNull Disposable d) {
	  
	                      }
	  
	                      @Override
	                      public void onNext(@NonNull Long aLong) {
	  
	                      }
	  
	                      @Override
	                      public void onError(@NonNull Throwable e) {
	  
	                      }
	  
	                      @Override
	                      public void onComplete() {
	  
	                      }
	                  });
	  ```
- ## 1、看返回的对象interval，谁调用的订阅方法subscribe
	- Observable.interval
		- ```java
		      @CheckReturnValue
		      @SchedulerSupport(SchedulerSupport.COMPUTATION)
		      @NonNull
		      public static Observable<Long> interval(long initialDelay, long period, @NonNull TimeUnit unit) {
		          return interval(initialDelay, period, unit, Schedulers.computation());
		      }
		  ```
		- 传入了scheduler =  Schedulers.computation
	- interval,返回了ObservableInterval对象
		- ```java
		      @CheckReturnValue
		      @NonNull
		      @SchedulerSupport(SchedulerSupport.CUSTOM)
		      public static Observable<Long> interval(long initialDelay, long period, @NonNull TimeUnit unit, @NonNull Scheduler scheduler) {
		          Objects.requireNonNull(unit, "unit is null");
		          Objects.requireNonNull(scheduler, "scheduler is null");
		  
		          return RxJavaPlugins.onAssembly(new ObservableInterval(Math.max(0L, initialDelay), Math.max(0L, period), unit, scheduler));
		      }
		  ```
- ## 2、看订阅流程，ObservableInterval的订阅最终调用
	- ObservableInterval  subscribeActual
		- ```java
		   @Override
		      public void subscribeActual(Observer<? super Long> observer) {
		          IntervalObserver is = new IntervalObserver(observer);
		          // 1、is 包装成了可取消的对象
		          observer.onSubscribe(is);
		  
		          Scheduler sch = scheduler;
		  
		          if (sch instanceof TrampolineScheduler) {
		              Worker worker = sch.createWorker();
		              is.setResource(worker);
		              worker.schedulePeriodically(is, initialDelay, period, unit);
		          } else {
		              // 2、is又是可执行的任务
		              Disposable d = sch.schedulePeriodicallyDirect(is, initialDelay, period, unit);
		              is.setResource(d);
		          }
		      }
		  ```
		- 1、工作原理是线程池执行定时任务，任务就是IntervalObserver，具体工作看其run方法就行
		- 2、包装成IntervalObserver  为 一个 Disposable 和 Runnable 。取消看其dispose方法就行
- ## 3、看其IntervalObserver，dispose
	- ```java
	          @Override
	          public void dispose() {
	              DisposableHelper.dispose(this);
	          }
	    //DisposableHelper ,就是让其 get方法变为 DISPOSED
	      public static boolean dispose(AtomicReference<Disposable> field) {
	          Disposable current = field.get();
	          Disposable d = DISPOSED;
	          if (current != d) {
	              current = field.getAndSet(d);
	              if (current != d) {
	                  if (current != null) {
	                      current.dispose();
	                  }
	                  return true;
	              }
	          }
	          return false;
	      }
	  ```
- ## 4、看其run方法
	- ```java
	          @Override
	          public void run() {
	              if (get() != DisposableHelper.DISPOSED) {
	                  downstream.onNext(count++);
	              }
	          }
	  ```
	- get 不为DISPOSED 才发送数据。则这个就是取消的原理。定义标记取消任务
-