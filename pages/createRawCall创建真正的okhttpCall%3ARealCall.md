- ## OkhttpCall.createRawCall
	- ```java
	    private okhttp3.Call createRawCall() throws IOException {
	      okhttp3.Call call = callFactory.newCall(requestFactory.create(args));
	      if (call == null) {
	        throw new NullPointerException("Call.Factory returned null.");
	      }
	      return call;
	    }
	  ```
- ## 寻找callFactory 是谁（OkhttpClient）
	- ## callFactory为OkhttpCall初始化的时候传入的
	  collapsed:: true
		- ```java
		    OkHttpCall(
		        RequestFactory requestFactory,
		        Object[] args,
		        okhttp3.Call.Factory callFactory,
		        Converter<ResponseBody, T> responseConverter) {
		      this.requestFactory = requestFactory;
		      this.args = args;
		      this.callFactory = callFactory;
		      this.responseConverter = responseConverter;
		    }
		  
		  ```
	- ## HttpServiceMethod.invoke 创建OkhttpCall传入的
	  collapsed:: true
		- ```java
		    @Override
		    final @Nullable ReturnT invoke(Object[] args) {
		      Call<ResponseT> call = new OkHttpCall<>(requestFactory, args, callFactory, responseConverter);
		      return adapt(call, args);
		    }
		  ```
	- ## CallAdapted 初始化时，调用了super传给 HttpServiceMethod的
	  collapsed:: true
		- ```java
		      CallAdapted(
		          RequestFactory requestFactory,
		          okhttp3.Call.Factory callFactory,
		          Converter<ResponseBody, ResponseT> responseConverter,
		          CallAdapter<ResponseT, ReturnT> callAdapter) {
		        super(requestFactory, callFactory, responseConverter);
		        this.callAdapter = callAdapter;
		      }
		  ```
	- ## HttpServiceMethod.parseAnnotations,创建CallAdapted 时传入的
	  collapsed:: true
		- ```java
		   okhttp3.Call.Factory callFactory = retrofit.callFactory;
		      if (!isKotlinSuspendFunction) {
		        return new CallAdapted<>(requestFactory, callFactory, responseConverter, callAdapter);
		      }
		  ```
	- ## 取的是retrofit.callFactory,而callFactory是build时赋值的
	  collapsed:: true
		- ```java
		      okhttp3.Call.Factory callFactory = this.callFactory;
		        if (callFactory == null) {
		          callFactory = new OkHttpClient();
		        }
		  
		  ```
	- ## [[#red]]==**总结是OkhttpClient**==
- ## 则看OkhttpClient.newCall
	- ```java
	    @Override public Call newCall(Request request) {
	      return RealCall.newRealCall(this, request, false /* for web socket */);
	    }
	  ```
- ##  RealCall.newRealCall
	- ```java
	    static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
	      // Safely publish the Call instance to the EventListener.
	      RealCall call = new RealCall(client, originalRequest, forWebSocket);
	      call.transmitter = new Transmitter(client, call);
	      return call;
	    }
	  ```
-