- # Observer接口
	- ## 接口
	  collapsed:: true
		- ```java
		  public interface Observer<T> {
		  
		      /**
		       * Provides the Observer with the means of cancelling (disposing) the
		       * connection (channel) with the Observable in both
		       * synchronous (from within {@link #onNext(Object)}) and asynchronous manner.
		       * @param d the Disposable instance whose {@link Disposable#dispose()} can
		       * be called anytime to cancel the connection
		       * @since 2.0
		       */
		      // 一订阅就先执行这个函数，
		      void onSubscribe(@NonNull Disposable d);
		  
		      /**
		       * Provides the Observer with a new item to observe.
		       * <p>
		       * The {@link Observable} may call this method 0 or more times.
		       * <p>
		       * The {@code Observable} will not call this method again after it calls either {@link #onComplete} or
		       * {@link #onError}.
		       *
		       * @param t
		       *          the item emitted by the Observable
		       */
		      void onNext(@NonNull T t);
		  
		      /**
		       * Notifies the Observer that the {@link Observable} has experienced an error condition.
		       * <p>
		       * If the {@link Observable} calls this method, it will not thereafter call {@link #onNext} or
		       * {@link #onComplete}.
		       *
		       * @param e
		       *          the exception encountered by the Observable
		       */
		      void onError(@NonNull Throwable e);
		  
		      /**
		       * Notifies the Observer that the {@link Observable} has finished sending push-based notifications.
		       * <p>
		       * The {@link Observable} will not call this method if it calls {@link #onError}.
		       */
		      void onComplete();
		  
		  }
		  
		  ```
	- ## 泛型T的作用？
	  collapsed:: true
		- 创建时候传什么类型，onNext回调函数就返回什么类型
		  collapsed:: true
			- ```java
			   // 终点
			                  new Observer<Bitmap>() {
			  
			                      // 订阅开始
			                      @Override
			                      public void onSubscribe(Disposable d) {
			                          // 预备 开始 要分发
			                          // TODO 第一步
			                          progressDialog = new ProgressDialog(DownloadActivity.this);
			                          progressDialog.setTitle("download run");
			                          progressDialog.show();
			                      }
			  
			                      // TODO 第四步
			                      // 拿到事件
			                      @Override
			                      public void onNext(Bitmap bitmap) {
			                          image.setImageBitmap(bitmap);
			                      }
			  
			                      // 错误事件
			                      @Override
			                      public void onError(Throwable e) {
			  
			                      }
			  
			                      // TODO 第五步
			                      // 完成事件
			                      @Override
			                      public void onComplete() {
			                          if (progressDialog != null)
			                              progressDialog.dismiss();
			                      }
			          });
			  ```
	- ## 函数作用
	  collapsed:: true
		- # onSubscribe
		  collapsed:: true
			- 订阅开始之前在主线程被回调：可以用来初始化操作，一订阅就执行这个
			- 里的参数Disposable（丢弃的意思）：  取消订阅 Disposable.dispose()
				- 作用1 ：就是可以用这个断开上下游联系，取消订阅，  【实际上是从下游观察者（SingleObserver）角度，丢弃你上游的被观察者。。出发点是由下游发起的 】
				- 作用2： 同时让上游停止生产的工作
		- # onComplete：事件结束
		- # onError：拿到上一个卡片的错误数据
		- # onNext：拿到上一个卡片留下来的数据