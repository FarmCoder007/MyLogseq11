title:: Map源码分析-rx2.0

- ## 代码
  collapsed:: true
	- ```java
	  public class SourceActivity3 extends AppCompatActivity {
	  
	      @Override
	      protected void onCreate(Bundle savedInstanceState) {
	          super.onCreate(savedInstanceState);
	  
	          // ObseravbleCreate  自定义source传递进去了 == source
	          Observable.create(
	  
	                  // 自定义source
	                  new ObservableOnSubscribe<String>() {
	                      @Override
	                      public void subscribe(ObservableEmitter<String> e) throws Exception {
	  						 // 发射器.onNext
	                          emitter.onNext("A");
	                      }
	          })
	  
	           //  ObseravbleCreate.map
	          .map(new Function<String, Bitmap>() {
	              @Override
	              public Bitmap apply(String s) throws Exception {
	                  return null;
	              }
	          })
	  
	          // ObservableMap.subscribe
	          .subscribe(
	  
	                  // 自定义观察者（终点）
	                  new Observer<Bitmap>() {
	                      @Override
	                      public void onSubscribe(Disposable d) {
	  
	                      }
	  
	                      @Override
	                      public void onNext(Bitmap bitmap) {
	  
	                      }
	  
	                      @Override
	                      public void onError(Throwable e) {
	  
	                      }
	  
	                      @Override
	                      public void onComplete() {
	  
	                      }
	                  });
	      }
	  
	  ```
- ## 主流程
  collapsed:: true
	- 1、Observable.create(）-》ObseravbleCreate 传入自定义source
	  id:: 64bfa0a5-7eac-440f-ae47-b3ab871430c1
	- 2、.map->为ObseravbleCreate.map 创建的为ObservableMap，传入的this为ObseravbleCreate
	  id:: 64bfa0c3-3cf5-42c6-9f57-4bab9431a774
		- ```java
		      public final <R> Observable<R> map(Function<? super T, ? extends R> mapper) {
		          ObjectHelper.requireNonNull(mapper, "mapper is null");
		          return RxJavaPlugins.onAssembly(new ObservableMap<T, R>(this, mapper));
		      }
		  ```
	- 3、.subscribe为ObservableMap的subscribe。下边开始源码分析
- ## 分析订阅流程分析
	- ## 1、ObservableMap.subscribe,没定义看父类的Observable.subscribe
	  id:: 64bf9c1e-e8b6-4bb1-b2f4-6c83412a6bea
	  collapsed:: true
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
	- ## 2、调用的为ObservableMap.subscribeActual
	  id:: 64bf9be6-53d4-419d-abc7-d3eb56155343
	  collapsed:: true
		- source为ObservableMap主流程第2项，创建的时候传入的ObseravbleCreate
		- ```java
		      @Override
		      public void subscribeActual(Observer<? super U> t) {
		          source.subscribe(new MapObserver<T, U>(t, function));
		      }
		  ```
		- 1、创建了MapObserver传入自定义观察者
		  id:: 64bf9db0-b44c-459d-a35c-1ce49c9693fb
		- 2、source（之前分析的ObseravbleCreate）的subscribe方法传入MapObserver
		  collapsed:: true
			- ObseravbleCreate没有subscribe还是看父类
			  id:: 64bfa295-4cfe-4cfb-bbec-b92057d69f1a
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
			- 实际调用的还是ObseravbleCreate中的subscribeActual，只不过这次传入的为MapObserver
	- ## 3、ObseravbleCreate.subscribeActual
	  collapsed:: true
		- 参数observer这次为传入的MapObserver
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
		- 1、创建发射器CreateEmitter，传入MapObserver
		- 2、调用MapObserver的onSubscribe方法
		- 3、调用source.subscribe 传入发射器：CreateEmitter
			- source为ObseravbleCreate创建传入的即：
			- ```java
			  // ObseravbleCreate  自定义source传递进去了 == source
			          Observable.create(
			  
			                  // 自定义source
			                  new ObservableOnSubscribe<String>() {
			                      @Override
			                      public void subscribe(ObservableEmitter<String> e) throws Exception {
			  						 // 发射器.onNext
			                          emitter.onNext("A");
			                      }
			          })
			  
			  ```
		- 4、调用的发射器的onNext函数 发送数据
	- ## 4、即调用CreateEmitter发射器内部持有MapObserver
	  collapsed:: true
		- ```java
		          @Override
		          public void onNext(T t) {
		              if (t == null) {
		                  onError(new NullPointerException("onNext called with null. Null values are generally not allowed in 2.x operators and sources."));
		                  return;
		              }
		              if (!isDisposed()) {
		                  observer.onNext(t);
		              }
		          }
		  ```
	- ## 5、即调用的为MapObserver.onNext做变换mapper.apply(t）
	  collapsed:: true
		- ```java
		  		@Override
		          public void onNext(T t) {
		              if (done) {
		                  return;
		              }
		  
		              if (sourceMode != NONE) {
		                  actual.onNext(null);
		                  return;
		              }
		  
		              U v;
		  
		              try {
		                  v = ObjectHelper.requireNonNull(mapper.apply(t), "The mapper function returned a null value.");
		              } catch (Throwable ex) {
		                  fail(ex);
		                  return;
		              }
		              actual.onNext(v);
		          }
		  ```
		- 1、mapper.apply(t）针对数据T类型做变换，返回v类型
		  collapsed:: true
			- ```java
			  public interface Function<T, R> {
			      /**
			       * Apply some calculation to the input value and return some other value.
			       * @param t the input value
			       * @return the output value
			       * @throws Exception on error
			       */
			      R apply(@NonNull T t) throws Exception;
			  }
			  ```
			- 真正的实现是 map（）函数传入的Function就是 里边的mapper
				- ```java
				          .map(new Function<String, Bitmap>() {
				              @Override
				              public Bitmap apply(String s) throws Exception {
				                  return null;
				              }
				          })
				            
				  public final <R> Observable<R> map(Function<? super T, ? extends R> mapper) {
				          ObjectHelper.requireNonNull(mapper, "mapper is null");
				          return RxJavaPlugins.onAssembly(new ObservableMap<T, R>(this, mapper));
				      }
				  ```
		- 2、变换完后actual.onNext 为传入自定义自定义观察者  observer
	- ## 6、回调自定义观察者observer的onNext结束回调