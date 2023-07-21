title:: call.enqueue网络请求过程分析

- ## 网络请求过程分析
  collapsed:: true
	- ```java
	  //step3
	  Call<SharedListBean> sharedListCall = sharedListService.getSharedList(2,1);
	  //step4
	  sharedListCall.enqueue(new Callback<SharedListBean>() {
	      @Override
	      public void onResponse(Call<SharedListBean> call, Response<SharedListBean> response{
	          if (response.isSuccessful()) {
	              System.out.println(response.body().toString());
	          }
	      }
	      @Override
	      public void onFailure(Call<SharedListBean> call, Throwable t) {
	          t.printStackTrace();
	      }
	  });
	  ```
- ## 由上可知sharedListCall默认为ExecutorCallBackCall，如果第三步getSharedList返回的是rxJava的observer，则sharedListCall为rxJava对应的
  collapsed:: true
	- 本次先看[[ExecutorCallBackCall]]的enqueue方法
	-
- ## ExecutorCallBackCall.enqueue,实际调用的是OkhttpCall的enqueue，
	- okhttpCall.enqueue
	  collapsed:: true
		- ```java
		  @Override
		    public void enqueue(final Callback<T> callback) {
		      Objects.requireNonNull(callback, "callback == null");
		  
		      okhttp3.Call call;
		      Throwable failure;
		  
		      synchronized (this) {
		        if (executed) throw new IllegalStateException("Already executed.");
		        executed = true;
		  
		        call = rawCall;
		        failure = creationFailure;
		        if (call == null && failure == null) {
		          try {
		            call = rawCall = createRawCall();
		          } catch (Throwable t) {
		            throwIfFatal(t);
		            failure = creationFailure = t;
		          }
		        }
		      }
		  
		      if (failure != null) {
		        callback.onFailure(this, failure);
		        return;
		      }
		  
		      if (canceled) {
		        call.cancel();
		      }
		  
		      call.enqueue(
		          new okhttp3.Callback() {
		            @Override
		            public void onResponse(okhttp3.Call call, okhttp3.Response rawResponse) {
		              Response<T> response;
		              try {
		                response = parseResponse(rawResponse);
		              } catch (Throwable e) {
		                throwIfFatal(e);
		                callFailure(e);
		                return;
		              }
		  
		              try {
		                callback.onResponse(OkHttpCall.this, response);
		              } catch (Throwable t) {
		                throwIfFatal(t);
		                t.printStackTrace(); // TODO this is not great
		              }
		            }
		  
		            @Override
		            public void onFailure(okhttp3.Call call, IOException e) {
		              callFailure(e);
		            }
		  
		            private void callFailure(Throwable e) {
		              try {
		                callback.onFailure(OkHttpCall.this, e);
		              } catch (Throwable t) {
		                throwIfFatal(t);
		                t.printStackTrace(); // TODO this is not great
		              }
		            }
		          });
		    }
		  ```
		-
	- ### 1、创建Okhttp 的realCall
		- ### [[createRawCall创建真正的okhttpCall:RealCall]]
	- ### 2、利用realCall去调用enqueue方法，
	- ### 3、realCall的回调中会parseResponse解析Okhttp的数据，回调给Retrofit，Retrofit切换主线程给上层
		- ### [[parseResponse]]