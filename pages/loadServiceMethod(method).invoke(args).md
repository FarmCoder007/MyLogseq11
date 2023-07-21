title:: loadServiceMethod(method).invoke(args)

- ## 背景
	- 由上一步可知相当于调用了CallAdapted 的 invoke方法，查看源码[[CallAdapted]]（上一步有） 没有invoke方法，则是在父类HttpServiceMethod 实现的
- ## 1、查看HttpServiceMethod.invoke方法
  collapsed:: true
	- ```java
	    @Override
	    final @Nullable ReturnT invoke(Object[] args) {
	      Call<ResponseT> call = new OkHttpCall<>(requestFactory, args, callFactory, responseConverter);
	      return adapt(call, args);
	    }
	  
	  ```
	- 1、创建了个[[OkHttpCall]]
	- 2、调用adapt，传入 Okhttpcall 和 方法入参，
		- ```java
		    protected abstract @Nullable ReturnT adapt(Call<ResponseT> call, Object[] args);
		  ```
		- 抽象方法，其实调用的是子类的CallAdapted的adapt方法
- ## 2、CallAdapted.adapt
  collapsed:: true
	- CallAdapted.adapt
		- ```java
		  static final class CallAdapted<ResponseT, ReturnT> extends HttpServiceMethod<ResponseT, ReturnT> {
		      private final CallAdapter<ResponseT, ReturnT> callAdapter;
		  
		      CallAdapted(
		          RequestFactory requestFactory,
		          okhttp3.Call.Factory callFactory,
		          Converter<ResponseBody, ResponseT> responseConverter,
		          CallAdapter<ResponseT, ReturnT> callAdapter) {
		        super(requestFactory, callFactory, responseConverter);
		        this.callAdapter = callAdapter;
		      }
		  
		      @Override
		      protected ReturnT adapt(Call<ResponseT> call, Object[] args) {
		        return callAdapter.adapt(call);
		      }
		    }
		  ```
	- 执行的成员变量的callAdapter.adapt(call)
- ## 3、[[追溯CallAdapted构造函数，寻找CallAdapter]]
- ## 4、2中CallAdapter为从Retrofit 的 DefaultCallAdapterFactory.get返回的实例，就是执行它的adapt
	- ```java
	  new CallAdapter<Object, Call<?>>() {
	        @Override
	        public Type responseType() {
	          return responseType;
	        }
	  
	        @Override
	        public Call<Object> adapt(Call<Object> call) {
	          return executor == null ? call : new ExecutorCallbackCall<>(executor, call);
	        }
	      };
	  ```
	- 结合2可以看出，将传入的OkHttpCall进行了包装返回了，[[ExecutorCallbackCall]]
-