- ## 代码
  collapsed:: true
	- ```java
	   Observable.create(
	  
	                  // 自定义source
	                  new ObservableOnSubscribe<String>() {
	                      @Override
	                      public void subscribe(ObservableEmitter<String> e) throws Exception {
	                          e.onNext("Derry");
	  
	                          Log.d(L.TAG, "自定义source: " + Thread.currentThread().getName());
	                      }
	                  })
	  
	                  // ObservableCreate。observeOn TODO 第二步骤
	                  .observeOn(
	  
	                          // TODO 第一步   主线程的 Handlnr
	                          AndroidSchedulers.mainThread()
	  
	                  )
	  
	                  // subsceOn(1)  // 会显示是这个线程的原因，上一层卡片是被1线程执行
	                  // subsceOn(2)
	                  // subsceOn(3)
	  
	                  // observeOn(A)
	                  // observeOn(B)
	                  // observeOn(C) // 终点是被C执行
	  
	  
	                  // ObservableObserveOn.subscribe
	                  .subscribe(
	  
	                          // 终点
	                          new Observer<String>() {
	                              @Override
	                              public void onSubscribe(Disposable d) {
	                                  Log.d(L.TAG, "onSubscribe: " +  Thread.currentThread().getName());
	                              }
	  
	                              @Override
	                              public void onNext(String s) {
	                                  Log.d(L.TAG, "onNext: " + Thread.currentThread().getName());
	                              }
	  
	                              @Override
	                              public void onError(Throwable e) {
	  
	                              }
	  
	                              @Override
	                              public void onComplete() {
	  
	                              }
	                          });
	  ```
- ## 一、先看操作符都返回啥
	- ## 1、AndroidSchedulers.mainThread()
		- ```java
		      public static Scheduler mainThread() {
		          return RxAndroidPlugins.onMainThreadScheduler(MAIN_THREAD);
		      }
		  ```
		- hook方法不用管，直接看MAIN_THREAD
		- ```java
		      private static final Scheduler MAIN_THREAD = RxAndroidPlugins.initMainThreadScheduler(
		              new Callable<Scheduler>() {
		                  @Override public Scheduler call() throws Exception {
		                      return MainHolder.DEFAULT;
		                  }
		              });
		  ```
	- ## 2、 MainHolder.DEFAULT
		- ```java
		      private static final class MainHolder {
		  
		          static final Scheduler DEFAULT = new HandlerScheduler(new Handler(Looper.getMainLooper()));
		      }
		  
		  ```
	- ## 3、创建了主线程的handler:HandlerScheduler,是个handler策略，
		- ```java
		  ```
- ## 二、走流程，看订阅触发者
	- ## 1、ObservableCreate。observeOn返回--->ObservableObserveOn
	  id:: 64bfdac6-dd97-4dc3-817e-b847c42cad64
		- ObservableCreate没定义方法看父类
		- ```java
		      @CheckReturnValue
		      @SchedulerSupport(SchedulerSupport.CUSTOM)
		      public final Observable<T> observeOn(Scheduler scheduler) {
		          return observeOn(scheduler, false, bufferSize());
		      }
		      @CheckReturnValue
		      @SchedulerSupport(SchedulerSupport.CUSTOM)
		      public final Observable<T> observeOn(Scheduler scheduler, boolean delayError, int bufferSize) {
		          ObjectHelper.requireNonNull(scheduler, "scheduler is null");
		          ObjectHelper.verifyPositive(bufferSize, "bufferSize");
		          return RxJavaPlugins.onAssembly(new ObservableObserveOn<T>(this, scheduler, delayError, bufferSize));
		      }
		  ```
		-
	- ## 2、则订阅走的ObservableObserveOn.subscribe
		- ```java
		   @Override
		      public final void subscribe(Observer<? super T> observer) {
		          ObjectHelper.requireNonNull(observer, "observer is null");
		          try {
		              observer = RxJavaPlugins.onSubscribe(this, observer);
		  
		              ObjectHelper.requireNonNull(observer, "Plugin returned null Observer");
		  
		              subscribeActual(observer);
		          } catch (NullPointerException e) { // NOPMD
		              throw e;
		          } catch (Throwable e) {
		              Exceptions.throwIfFatal(e);
		              // can't call onError because no way to know if a Disposable has been set or not
		              // can't call onSubscribe because the call might have set a Subscription already
		              RxJavaPlugins.onError(e);
		  
		              NullPointerException npe = new NullPointerException("Actually not, but can't throw other exceptions due to RS");
		              npe.initCause(e);
		              throw npe;
		          }
		      }
		  ```
	- ## 3、实际走ObservableObserveOn的subscribeActual
		- ```java
		      @Override
		      protected void subscribeActual(Observer<? super T> observer) {
		          if (scheduler instanceof TrampolineScheduler) {
		              source.subscribe(observer);
		          } else {
		              Scheduler.Worker w = scheduler.createWorker();
		  
		              source.subscribe(new ObserveOnObserver<T>(observer, w, delayError, bufferSize));
		          }
		      }
		  ```
		- source 和scheduler 为ObservableObserveOn创建时传入的，看1
		  id:: 64bfdc0e-fcfb-4665-89ec-485e573f6571
		- source  = ObservableCreate   scheduler = HandlerScheduler
		  id:: 64bfdca2-740c-4908-94a7-e296e8398a00
		-
	- ## 4、HandlerScheduler。createWorker
		- ```java
		      @Override
		      public Worker createWorker() {
		          return new HandlerWorker(handler);
		      }
		  ```
		- ```java
		  
		          HandlerWorker(Handler handler) {
		              this.handler = handler;
		          }
		  ```
		- 传入主线程的handler
	- ## 5、source.subscribe(new ObserveOnObserver
	  id:: 64bfdd4a-ad0f-48ed-a5c9-c97112fa89ef
		- 1、创建了ObserveOnObserver 传入了worker 和 自定义的终点observer
		- 2、调用source.subscribe 即 走ObservableCreate    的subscribeActual方法，传入包装的ObserveOnObserver
			- ```java
			      @Override
			      protected void subscribeActual(Observer<? super T> observer) {
			          CreateEmitter<T> parent = new CreateEmitter<T>(observer);
			          observer.onSubscribe(parent);
			  
			          try {
			              source.subscribe(parent);
			          } catch (Throwable ex) {
			              Exceptions.throwIfFatal(ex);
			              parent.onError(ex);
			          }
			      }
			  ```
			- 1、创建发射器CreateEmitter，传入ObserveOnObserver
			- 2、调用ObserveOnObserver 的onSubscribe传入发射器
			- 3、调用自定义source的subscribe方法传入 ObserveOnObserver
				- 自定义source是
					- ```java
					  Observable.create(
					  
					                  // 自定义source
					                  new ObservableOnSubscribe<String>() {
					                      @Override
					                      public void subscribe(ObservableEmitter<String> e) throws Exception {
					                          e.onNext("Derry");
					  
					                          Log.d(L.TAG, "自定义source: " + Thread.currentThread().getName());
					                      }
					          })
					  ```
			- 4、e.onNext("Derry");为ObserveOnObserver .onNext
	- ## 6、ObserveOnObserver .onNext
		- ```java
		         @Override
		          public void onNext(T t) {
		              if (done) {
		                  return;
		              }
		  
		              if (sourceMode != QueueDisposable.ASYNC) {
		                  queue.offer(t);
		              }
		              schedule();
		          }
		  ```
		- 要发送的数据加入队列，  ObserveOnObserver 还是个Runnable
		- 执行调度schedule
	- ## 7、schedule,worker为ObserveOnObserver 创建时传入的HandlerWorker
		- ```java
		       void schedule() {
		              if (getAndIncrement() == 0) {
		                  worker.schedule(this);
		              }
		          }
		  ```
	- ## 8、HandlerWorker调度这个 ObserveOnObserver  Runnable
		- 先父类
			- ```java
			      @NonNull
			          public Disposable schedule(@NonNull Runnable run) {
			              return schedule(run, 0L, TimeUnit.NANOSECONDS);
			          }
			  ```
		- ```java
		    
		  @Override
		          public Disposable schedule(Runnable run, long delay, TimeUnit unit) {
		              if (run == null) throw new NullPointerException("run == null");
		              if (unit == null) throw new NullPointerException("unit == null");
		  
		              if (disposed) {
		                  return Disposables.disposed();
		              }
		  
		              run = RxJavaPlugins.onSchedule(run);
		  
		              ScheduledRunnable scheduled = new ScheduledRunnable(handler, run);
		  
		              Message message = Message.obtain(handler, scheduled);
		              message.obj = this; // Used as token for batch disposal of this worker's runnables.
		  
		              handler.sendMessageDelayed(message, Math.max(0L, unit.toMillis(delay)));
		  
		              // Re-check disposed state for removing in case we were racing a call to dispose().
		              if (disposed) {
		                  handler.removeCallbacks(scheduled);
		                  return Disposables.disposed();
		              }
		  
		              return scheduled;
		          }
		  ```
	- ## 9、最终通过handler 发送任务到[[#red]]==**主线程**==接收，看run干啥事
		- ```java
		   @Override
		          public void run() {
		              if (outputFused) {
		                  drainFused();
		              } else {
		                  drainNormal();
		              }
		          }
		  ```
		- actual为实际的观察者对象
		- ```java
		     void drainNormal() {
		              int missed = 1;
		  
		              final SimpleQueue<T> q = queue;
		              final Observer<? super T> a = actual;
		  
		              for (;;) {
		                  if (checkTerminated(done, q.isEmpty(), a)) {
		                      return;
		                  }
		  
		                  for (;;) {
		                      boolean d = done;
		                      T v;
		  
		                      try {
		                          v = q.poll();
		                      } catch (Throwable ex) {
		                          Exceptions.throwIfFatal(ex);
		                          s.dispose();
		                          q.clear();
		                          a.onError(ex);
		                          worker.dispose();
		                          return;
		                      }
		                      boolean empty = v == null;
		  
		                      if (checkTerminated(d, empty, a)) {
		                          return;
		                      }
		  
		                      if (empty) {
		                          break;
		                      }
		  
		                      a.onNext(v);
		                  }
		  
		                  missed = addAndGet(-missed);
		                  if (missed == 0) {
		                      break;
		                  }
		              }
		          }
		  ```