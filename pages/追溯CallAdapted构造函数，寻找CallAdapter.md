- HttpServiceMethod.parseAnnotations，里创建了CallAdapted，传入了CallAdapter，CallAdapter是
	- ```java
	  CallAdapter<ResponseT, ReturnT> callAdapter =
	          createCallAdapter(retrofit, method, adapterType, annotations);
	  ```
- HttpServiceMethod.createCallAdapter
	- ```java
	    private static <ResponseT, ReturnT> CallAdapter<ResponseT, ReturnT> createCallAdapter(
	        Retrofit retrofit, Method method, Type returnType, Annotation[] annotations) {
	      try {
	        //noinspection unchecked
	        return (CallAdapter<ResponseT, ReturnT>) retrofit.callAdapter(returnType, annotations);
	      } catch (RuntimeException e) { // Wide exception range because factories are user code.
	        throw methodError(method, e, "Unable to create call adapter for %s", returnType);
	      }
	    }
	  ```
- retrofit.callAdapter
	- ```java
	    public CallAdapter<?, ?> callAdapter(Type returnType, Annotation[] annotations) {
	      return nextCallAdapter(null, returnType, annotations);
	    }
	  ```
- retrofit.nextCallAdapter：[[#red]]==**根据返回值returnType，从callAdapterFactories 返回具体的callAdapter**==
	- 代码
		- ```java
		    public CallAdapter<?, ?> nextCallAdapter(
		        @Nullable CallAdapter.Factory skipPast, Type returnType, Annotation[] annotations) {
		      Objects.requireNonNull(returnType, "returnType == null");
		      Objects.requireNonNull(annotations, "annotations == null");
		  
		      int start = callAdapterFactories.indexOf(skipPast) + 1;
		      for (int i = start, count = callAdapterFactories.size(); i < count; i++) {
		        CallAdapter<?, ?> adapter = callAdapterFactories.get(i).get(returnType, annotations, this);
		        if (adapter != null) {
		          return adapter;
		        }
		      }
		  
		      StringBuilder builder =
		          new StringBuilder("Could not locate call adapter for ").append(returnType).append(".\n");
		      if (skipPast != null) {
		        builder.append("  Skipped:");
		        for (int i = 0; i < start; i++) {
		          builder.append("\n   * ").append(callAdapterFactories.get(i).getClass().getName());
		        }
		        builder.append('\n');
		      }
		      builder.append("  Tried:");
		      for (int i = start, count = callAdapterFactories.size(); i < count; i++) {
		        builder.append("\n   * ").append(callAdapterFactories.get(i).getClass().getName());
		      }
		      throw new IllegalArgumentException(builder.toString());
		    }
		  ```
	- [[#red]]==**根据返回值returnType，从callAdapterFactories 返回具体的callAdapter**== .callAdapterFactories 是在最开始里Retrofit build 构造实例时，初始化的，传入了默认的。这里返回call，应该是初始化时默认的那个DefaultCallAdapterFactory。详解见Retrofit的 build 流程
	- callAdapterFactories.get(i).get 为DefaultCallAdapterFactory.get()方法 获取CallAdapter
		- ```java
		  public @Nullable CallAdapter<?, ?> get(
		        Type returnType, Annotation[] annotations, Retrofit retrofit) {
		      if (getRawType(returnType) != Call.class) {
		        return null;
		      }
		      if (!(returnType instanceof ParameterizedType)) {
		        throw new IllegalArgumentException(
		            "Call return type must be parameterized as Call<Foo> or Call<? extends Foo>");
		      }
		      final Type responseType = Utils.getParameterUpperBound(0, (ParameterizedType) returnType);
		  
		      final Executor executor =
		          Utils.isAnnotationPresent(annotations, SkipCallbackExecutor.class)
		              ? null
		              : callbackExecutor;
		  
		      return new CallAdapter<Object, Call<?>>() {
		        @Override
		        public Type responseType() {
		          return responseType;
		        }
		  
		        @Override
		        public Call<Object> adapt(Call<Object> call) {
		          return executor == null ? call : new ExecutorCallbackCall<>(executor, call);
		        }
		      };
		    }
		  ```