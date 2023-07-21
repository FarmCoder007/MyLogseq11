- 代码
	- ```java
	      static final class ExecutorCallbackCall<T> implements Call<T> {
	          final Executor callbackExecutor;
	          final Call<T> delegate;
	  
	          ExecutorCallbackCall(Executor callbackExecutor, Call<T> delegate) {
	              this.callbackExecutor = callbackExecutor;
	              this.delegate = delegate;
	          }
	  
	          public void enqueue(final Callback<T> callback) {
	              Objects.requireNonNull(callback, "callback == null");
	              this.delegate.enqueue(new Callback<T>() {
	                  public void onResponse(Call<T> call, Response<T> response) {
	                      ExecutorCallbackCall.this.callbackExecutor.execute(() -> {
	                          if (ExecutorCallbackCall.this.delegate.isCanceled()) {
	                              callback.onFailure(ExecutorCallbackCall.this, new IOException("Canceled"));
	                          } else {
	                              callback.onResponse(ExecutorCallbackCall.this, response);
	                          }
	  
	                      });
	                  }
	  
	                  public void onFailure(Call<T> call, Throwable t) {
	                      ExecutorCallbackCall.this.callbackExecutor.execute(() -> {
	                          callback.onFailure(ExecutorCallbackCall.this, t);
	                      });
	                  }
	              });
	          }
	  
	          public boolean isExecuted() {
	              return this.delegate.isExecuted();
	          }
	  
	          public Response<T> execute() throws IOException {
	              return this.delegate.execute();
	          }
	  
	          public void cancel() {
	              this.delegate.cancel();
	          }
	  
	          public boolean isCanceled() {
	              return this.delegate.isCanceled();
	          }
	  
	          public Call<T> clone() {
	              return new ExecutorCallbackCall(this.callbackExecutor, this.delegate.clone());
	          }
	  
	          public Request request() {
	              return this.delegate.request();
	          }
	  
	          public Timeout timeout() {
	              return this.delegate.timeout();
	          }
	      }
	  ```
- 作用：
	- 对OkHttpCall进行了包装，把后台线程 切到 主线程    可以直接 写ui