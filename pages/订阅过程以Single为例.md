- ## 示例代码
  collapsed:: true
	- ```java
	          Single.just("1").subscribe(new SingleObserver<String>() {
	              @Override
	              public void onSubscribe(@NonNull Disposable d) {
	                  
	              }
	  
	              @Override
	              public void onSuccess(@NonNull String s) {
	  
	              }
	  
	              @Override
	              public void onError(@NonNull Throwable e) {
	  
	              }
	          });
	  ```
- ## 角色分三部分
	- ## 被观察者observable
	  collapsed:: true
		- ## 1、Single.just("1")
			- ```java
			      @CheckReturnValue
			      @SchedulerSupport(SchedulerSupport.NONE)
			      @NonNull
			      public static <@NonNull T> Single<T> just(T item) {
			          Objects.requireNonNull(item, "item is null");
			          return RxJavaPlugins.onAssembly(new SingleJust<>(item));
			      }
			  ```
			- onAssembly之前介绍的hook方法 则返回的为SingleJust对象，传入操作数据
		- ## 2、SingleJust
			- ```java
			  public final class SingleJust<T> extends Single<T> {
			  
			      final T value;
			  
			      public SingleJust(T value) {
			          this.value = value;
			      }
			  
			      @Override
			      protected void subscribeActual(SingleObserver<? super T> observer) {
			          observer.onSubscribe(Disposable.disposed());
			          observer.onSuccess(value);
			      }
			  
			  }
			  ```
	- ## 自定义观察者SingleObserver
	  collapsed:: true
		- 就是传入了接口实现类
		- ```java
		  public interface SingleObserver<@NonNull T> {
		  
		      /**
		       * Provides the {@link SingleObserver} with the means of cancelling (disposing) the
		       * connection (channel) with the Single in both
		       * synchronous (from within {@code onSubscribe(Disposable)} itself) and asynchronous manner.
		       * @param d the Disposable instance whose {@link Disposable#dispose()} can
		       * be called anytime to cancel the connection
		       * @since 2.0
		       */
		      void onSubscribe(@NonNull Disposable d);
		  
		      /**
		       * Notifies the {@link SingleObserver} with a single item and that the {@link Single} has finished sending
		       * push-based notifications.
		       * <p>
		       * The {@code Single} will not call this method if it calls {@link #onError}.
		       *
		       * @param t
		       *          the item emitted by the {@code Single}
		       */
		      void onSuccess(@NonNull T t);
		  
		      /**
		       * Notifies the {@link SingleObserver} that the {@link Single} has experienced an error condition.
		       * <p>
		       * If the {@code Single} calls this method, it will not thereafter call {@link #onSuccess}.
		       *
		       * @param e
		       *          the exception encountered by the {@code Single}
		       */
		      void onError(@NonNull Throwable e);
		  }
		  ```
	- ## 订阅过程subscribe
	  collapsed:: true
		- 那么看订阅过程
		- ## 1、Single.just("1").subscribe其实调用的是SingleJust.subscribe
		  collapsed:: true
			- 但是SingleJust没有订阅的方法则 拿父类Single的
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
				- 内部调用subscribeActual，为子类的实现方法
		- ## 2、SingleJust.subscribeActual,传入自定义观察者
			- ```java
			      @Override
			      protected void subscribeActual(SingleObserver<? super T> observer) {
			          observer.onSubscribe(Disposable.disposed());
			          observer.onSuccess(value);
			      }
			  ```
			- 1、调用自定义观察者的开始订阅方法
			- 2、调用onSuccess方法 传递数据。则进行回调
		- ## 3、SingleObserver相应的方法进行接收