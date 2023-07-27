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
	  
	          // ObservbaleCreate.subscribeOn
	          // TODO 第二步     new IOScheduler ---> 线程池 传递进去
	          .subscribeOn(
	  
	                  // TODO 第一步  到底干了什么 （ new IOScheduler ---> 线程池）
	                  Schedulers.io() // 耗时读取的异步
	  
	                  // Schedulers.newThread() // 开启新线程
	  
	          )
	  
	           // A线程. subscribe
	           //  ObservableSubscribeOn.subscribe
	          .subscribe(
	  
	                  // 终点
	                  new Observer<String>() {
	                      @Override
	                      public void onSubscribe(Disposable d) {
	  
	                          Disposable disposable = d;
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
- ## 一、Schedulers.io---> 创建的IOScheduler 线程池
  collapsed:: true
	- ## 1、**Schedulers**.io() // 耗时读取的异步
	  collapsed:: true
		- ```java
		      @NonNull
		      public static Scheduler io() {
		          return RxJavaPlugins.onIoScheduler(IO);
		      }
		  ```
		- 又一个hook函数,hook Scheduler的
		- ```java
		  public static Scheduler onIoScheduler(@NonNull Scheduler defaultScheduler) {
		      Function<? super Scheduler, ? extends Scheduler> f = onIoHandler;
		      if (f == null) {
		          return defaultScheduler;
		      }
		      return apply(f, defaultScheduler);
		  }
		  ```
	- ## 2、看IO
	  collapsed:: true
		- ```java
		      @NonNull
		      static final Scheduler IO;
		  
		         static {
		          SINGLE = RxJavaPlugins.initSingleScheduler(new SingleTask());
		  
		          COMPUTATION = RxJavaPlugins.initComputationScheduler(new ComputationTask());
		  
		          IO = RxJavaPlugins.initIoScheduler(new IOTask());
		  
		          TRAMPOLINE = TrampolineScheduler.instance();
		  
		          NEW_THREAD = RxJavaPlugins.initNewThreadScheduler(new NewThreadTask());
		      }
		  ```
	- ## 3、RxJavaPlugins.initIoScheduler(new IOTask());
	  id:: 64bfcb42-1856-4c0b-8d2c-49f1c9365f5c
	  collapsed:: true
		- 又是一个hook函数
		- ```java
		      @NonNull
		      public static Scheduler initIoScheduler(@NonNull Callable<Scheduler> defaultScheduler) {
		          ObjectHelper.requireNonNull(defaultScheduler, "Scheduler Callable can't be null");
		          Function<? super Callable<Scheduler>, ? extends Scheduler> f = onInitIoHandler;
		          if (f == null) {
		              return callRequireNonNull(defaultScheduler);
		          }
		          return applyRequireNonNull(f, defaultScheduler);
		      }
		  
		  ```
		- 则只看IOTask
	- ## 4、IOTask
	  collapsed:: true
		- ```java
		      static final class IOTask implements Callable<Scheduler> {
		          @Override
		          public Scheduler call() throws Exception {
		              return IoHolder.DEFAULT;
		          }
		      }
		  ```
		- ```java
		      static final class IoHolder {
		          static final Scheduler DEFAULT = new IoScheduler();
		      }
		  ```
	- ## 5、IoScheduler
	  collapsed:: true
		- 构造函数
		- ```java
		      static final RxThreadFactory WORKER_THREAD_FACTORY;    
		  	public IoScheduler() {
		          // 传入一个 线程工厂
		          this(WORKER_THREAD_FACTORY);
		      }
		  
		  	   static {
		   		WORKER_THREAD_FACTORY = new RxThreadFactory(WORKER_THREAD_NAME_PREFIX, priority);
		      }
		  ```
		- ```java
		      public IoScheduler(ThreadFactory threadFactory) {
		          this.threadFactory = threadFactory;
		          this.pool = new AtomicReference<CachedWorkerPool>(NONE);
		          start();
		      }
		   
		  
		  ```
	- ## 6、start();
	  collapsed:: true
		- ```java
		      @Override
		      public void start() {
		          // 缓存池
		          CachedWorkerPool update = new CachedWorkerPool(KEEP_ALIVE_TIME, KEEP_ALIVE_UNIT, threadFactory);
		          if (!pool.compareAndSet(NONE, update)) {
		              update.shutdown();
		          }
		      }
		  ```
	- ## 7、shutdown
	  collapsed:: true
		- ```java
		          void shutdown() {
		              allWorkers.dispose();
		              if (evictorTask != null) {
		                  evictorTask.cancel(true);
		              }
		              if (evictorService != null) {
		                  evictorService.shutdownNow();
		              }
		          }
		  ```
		- evictorService 线程池
		- ```java
		   private final ScheduledExecutorService evictorService;
		  ```
	- ## 总结 （new IOScheduler ---> 线程池）
		-
- ## 二、subscribeOn（IOScheduler ）
  collapsed:: true
	- this 实际为 ObservbaleCreate
	- ```java
	      @CheckReturnValue
	      @SchedulerSupport(SchedulerSupport.CUSTOM)
	      public final Observable<T> subscribeOn(Scheduler scheduler) {
	          ObjectHelper.requireNonNull(scheduler, "scheduler is null");
	          return RxJavaPlugins.onAssembly(new ObservableSubscribeOn<T>(this, scheduler));
	      }
	  ```
	- ## ObservableSubscribeOn，scheduler 为IOScheduler
		- ```java
		      public ObservableSubscribeOn(ObservableSource<T> source, Scheduler scheduler) {
		          super(source);
		          this.scheduler = scheduler;
		      }
		  ```
	- 了解了参数后，再看整个流程
- ## 三、看订阅怎么触发的subscribe
  collapsed:: true
	- subscribeOn 返回的ObservableSubscribeOn
	- 则看ObservableSubscribeOn的subscribe
	  id:: 64bfd1dc-8503-462b-a75b-e2ea0ffb7eba
		- 父类的subscribe,内部调用子类的ObservableSubscribeOn的subscribeActual
		- ```java
		  @SchedulerSupport(SchedulerSupport.NONE)
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
- ## 四、ObservableSubscribeOn的subscribeActual
  collapsed:: true
	- ```java
	      @Override
	      public void subscribeActual(final Observer<? super T> s) {
	          final SubscribeOnObserver<T> parent = new SubscribeOnObserver<T>(s);
	  
	          s.onSubscribe(parent);
	  
	          parent.setDisposable(scheduler.scheduleDirect(new SubscribeTask(parent)));
	      }
	  
	  ```
	- 1、创建SubscribeOnObserver，传入了自定义的观察者Observer
	  id:: 64bfd267-6d71-4a91-8a3e-d4061727dedf
	- 2、调用了自定义Observer的onSubscribe
	- 3、SubscribeOnObserver.setDisposable
- ## 看任务是干嘛的
  collapsed:: true
	- ## 五、new SubscribeTask(parent))，就是个Runnable任务
	  collapsed:: true
		- 传入SubscribeOnObserver
		- ```java
		      final class SubscribeTask implements Runnable {
		          private final SubscribeOnObserver<T> parent;
		  
		          SubscribeTask(SubscribeOnObserver<T> parent) {
		              this.parent = parent;
		          }
		  
		          @Override
		          public void run() {
		              source.subscribe(parent);
		          }
		      }
		  ```
		- 看run,source 为 ObservbaleCreate，见SubscribeOnObserver创建分析
			- source.subscribe(parent);  为 ObservbaleCreate
	- ## [[#red]]==**从这里进入io线程，引入从任务里执行的**==
	- ## 六、ObservbaleCreate.subscribe,老套路，调用父类方法
		- ```java
		  @SchedulerSupport(SchedulerSupport.NONE)
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
		- 内部调用 subscribeActual(observer); 传入 SubscribeOnObserver
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
		- 1、创建发射器CreateEmitter，传入SubscribeOnObserver
		  id:: 64bfd3ff-3710-4b3f-91f0-3f2db4aeff84
		- 2、调用SubscribeOnObserver的onSubscribe传入发射器
		- 3、调用自定义source的subscribe方法传入 SubscribeOnObserver
		  collapsed:: true
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
		- 4、e.onNext("Derry");为SubscribeOnObserver.onNext
	- ## 七、发送数据SubscribeOnObserver.onnext()
	  collapsed:: true
		- ```java
		          @Override
		          public void onNext(T t) {
		              actual.onNext(t);
		          }
		  ```
		- actual 为真实的自定义观察者 Observer,则回调完成，
		-
- ## 怎么交给线程池的
  collapsed:: true
	- ## scheduler.scheduleDirect
	  collapsed:: true
		- ```java
		    public Disposable scheduleDirect(@NonNull Runnable run) {
		          return scheduleDirect(run, 0L, TimeUnit.NANOSECONDS);
		      }
		  ```
		- ```java
		      public Disposable scheduleDirect(@NonNull Runnable run, long delay, @NonNull TimeUnit unit) {
		          final Worker w = createWorker();
		  
		          final Runnable decoratedRun = RxJavaPlugins.onSchedule(run);
		  
		          DisposeTask task = new DisposeTask(decoratedRun, w);
		  
		          w.schedule(task, delay, unit);
		  
		          return task;
		      }
		  ```
	- ## 包装了下交给了 Worker.schedule
	  id:: 64bfd74b-cb3e-4408-8bad-04753d08311b
	  collapsed:: true
		- Work在ioSchedule创建的地方
		- ```java
		      @NonNull
		      @Override
		      public Worker createWorker() {
		          return new EventLoopWorker(pool.get());
		      }
		  ```
	- ## 执行EventLoopWorker.schedule
	  collapsed:: true
		- ```java
		          @NonNull
		          @Override
		          public Disposable schedule(@NonNull Runnable action, long delayTime, @NonNull TimeUnit unit) {
		              if (tasks.isDisposed()) {
		                  // don't schedule, we are unsubscribed
		                  return EmptyDisposable.INSTANCE;
		              }
		  
		              return threadWorker.scheduleActual(action, delayTime, unit, tasks);
		          }
		  ```
	- ## 交给线程池执行
	  collapsed:: true
		- ```java
		    public ScheduledRunnable scheduleActual(final Runnable run, long delayTime, @NonNull TimeUnit unit, @Nullable DisposableContainer parent) {
		          Runnable decoratedRun = RxJavaPlugins.onSchedule(run);
		  
		          ScheduledRunnable sr = new ScheduledRunnable(decoratedRun, parent);
		  
		          if (parent != null) {
		              if (!parent.add(sr)) {
		                  return sr;
		              }
		          }
		  
		          Future<?> f;
		          try {
		              if (delayTime <= 0) {
		                  f = executor.submit((Callable<Object>)sr);
		              } else {
		                  f = executor.schedule((Callable<Object>)sr, delayTime, unit);
		              }
		              sr.setFuture(f);
		          } catch (RejectedExecutionException ex) {
		              if (parent != null) {
		                  parent.remove(sr);
		              }
		              RxJavaPlugins.onError(ex);
		          }
		  
		          return sr;
		      }
		  ```
-