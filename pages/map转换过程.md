## 代码
	- ```java
	  Single.just("1")// SingleJust.map
	                  .map(new Function<String, Integer>() {
	  
	                      @Override
	                      public Integer apply(String s) throws Throwable {
	                          return Integer.parseInt(s);
	                      }
	                  }).subscribe(new SingleObserver<Integer>() {// Singmap
	                      @Override
	                      public void onSubscribe(@NonNull Disposable d) {
	  
	                      }
	  
	                      @Override
	                      public void onSuccess(@NonNull Integer s) {
	  
	                      }
	  
	                      @Override
	                      public void onError(@NonNull Throwable e) {
	  
	                      }
	                  });
	  ```
- ## 成员介绍
	- 1、Single.just("1") 返回的是SIngleJust 被观察者
	- 2、map相当于  SIngleJust.map  返回SingleMap对象
		- ```java
		      @CheckReturnValue
		      @NonNull
		      @SchedulerSupport(SchedulerSupport.NONE)
		      public final <@NonNull R> Single<R> map(@NonNull Function<? super T, ? extends R> mapper) {
		          Objects.requireNonNull(mapper, "mapper is null");
		          return RxJavaPlugins.onAssembly(new SingleMap<>(this, mapper));
		      }
		  ```
	- 3、那么订阅函数相当于是SingleMap.subscribe传入自定义 观察者 SingleObserver
- ## 看订阅流程解析
	- ### 1、则直接看SingleMap.subscribe.只要是订阅都是老套路，最终 调用子类的
		- ```java
		  @SchedulerSupport(SchedulerSupport.NONE)
		      @Override
		      public final void subscribe(@NonNull SingleObserver<? super T> observer) {
		          Objects.requireNonNull(observer, "observer is null");
		  
		          observer = RxJavaPlugins.onSubscribe(this, observer);
		  
		          Objects.requireNonNull(observer, "The RxJavaPlugins.onSubscribe hook returned a null SingleObserver. Please check the handler provided to RxJavaPlugins.setOnSingleSubscribe for invalid null returns. Further reading: https://github.com/ReactiveX/RxJava/wiki/Plugins");
		  
		          try {
		              subscribeActual(observer);
		          } catch (NullPointerException ex) {
		              throw ex;
		          } catch (Throwable ex) {
		              Exceptions.throwIfFatal(ex);
		              NullPointerException npe = new NullPointerException("subscribeActual failed");
		              npe.initCause(ex);
		              throw npe;
		          }
		      }
		  ```
	- ### 2、调用子类的即 SingleMap函数的subscribeActual
		- ```java
		      @Override
		      protected void subscribeActual(final SingleObserver<? super R> t) {
		          source.subscribe(new MapSingleObserver<T, R>(t, mapper));
		      }
		  ```
		- source为  SIngleJust.map创建 SIngleMap时传入的 SingleJust，mapper为传入的function函数
		- 1、创建MapSingleObserver 传入自定义观察者SingleObserver  和 转换函数mapper
		- 2、则调用的SIngleJust 的订阅方法 传入包装观察者
	- ### 3、只要是订阅都是一样的套路，直接看SIngleJust.subscribeActual方法
		- ```java
		      @Override
		      protected void subscribeActual(SingleObserver<? super T> observer) {
		          observer.onSubscribe(Disposable.disposed());
		          observer.onSuccess(value);
		      }
		  ```
		- 传入的observer为包装的里边包裹者 转换函数，和 自定义观察者
		- 1、调用MapSingleObserver的 开始订阅方法onSubscribe
			- ```java
			         @Override
			          public void onSubscribe(Disposable d) {
			              t.onSubscribe(d);
			          }
			  ```
			- t就是创建对象时，传入的自定义观察者，这里由此回调回去
		- 2、调用MapSingleObserver. onSuccess回调参数
			- ```java
			   @Override
			          public void onSuccess(T value) {
			              R v;
			              try {
			                  v = Objects.requireNonNull(mapper.apply(value), "The mapper function returned a null value.");
			              } catch (Throwable e) {
			                  Exceptions.throwIfFatal(e);
			                  onError(e);
			                  return;
			              }
			  
			              t.onSuccess(v);
			          }
			  ```
			- 1、调用转换函数的apply方法 将参数转换为返回值 v
			- 2、调用自定义观察者的onSuccess方法回调转换后的参数
- # [[订阅发生时，流转过程]]