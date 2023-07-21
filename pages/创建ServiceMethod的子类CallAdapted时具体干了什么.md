- ## HttpServiceMethod.java  parseAnnotations
  collapsed:: true
	- ```java
	  static <ResponseT, ReturnT> HttpServiceMethod<ResponseT, ReturnT> parseAnnotations(
	        Retrofit retrofit, Method method, RequestFactory requestFactory) {
	      boolean isKotlinSuspendFunction = requestFactory.isKotlinSuspendFunction;
	      boolean continuationWantsResponse = false;
	      boolean continuationBodyNullable = false;
	  
	      Annotation[] annotations = method.getAnnotations();
	      Type adapterType;
	      if (isKotlinSuspendFunction) {
	        Type[] parameterTypes = method.getGenericParameterTypes();
	        Type responseType =
	            Utils.getParameterLowerBound(
	                0, (ParameterizedType) parameterTypes[parameterTypes.length - 1]);
	        if (getRawType(responseType) == Response.class && responseType instanceof ParameterizedType) {
	          // Unwrap the actual body type from Response<T>.
	          responseType = Utils.getParameterUpperBound(0, (ParameterizedType) responseType);
	          continuationWantsResponse = true;
	        } else {
	          // TODO figure out if type is nullable or not
	          // Metadata metadata = method.getDeclaringClass().getAnnotation(Metadata.class)
	          // Find the entry for method
	          // Determine if return type is nullable or not
	        }
	  
	        adapterType = new Utils.ParameterizedTypeImpl(null, Call.class, responseType);
	        annotations = SkipCallbackExecutorImpl.ensurePresent(annotations);
	      } else {
	        adapterType = method.getGenericReturnType();
	      }
	      
	      CallAdapter<ResponseT, ReturnT> callAdapter =
	          createCallAdapter(retrofit, method, adapterType, annotations);
	      Type responseType = callAdapter.responseType();
	      if (responseType == okhttp3.Response.class) {
	        throw methodError(
	            method,
	            "'"
	                + getRawType(responseType).getName()
	                + "' is not a valid response body type. Did you mean ResponseBody?");
	      }
	      if (responseType == Response.class) {
	        throw methodError(method, "Response must include generic type (e.g., Response<String>)");
	      }
	      // TODO support Unit for Kotlin?
	      if (requestFactory.httpMethod.equals("HEAD") && !Void.class.equals(responseType)) {
	        throw methodError(method, "HEAD method must use Void as response type.");
	      }
	  
	      Converter<ResponseBody, ResponseT> responseConverter =
	          createResponseConverter(retrofit, method, responseType);
	  
	      okhttp3.Call.Factory callFactory = retrofit.callFactory;
	      if (!isKotlinSuspendFunction) {
	        return new CallAdapted<>(requestFactory, callFactory, responseConverter, callAdapter);
	      } else if (continuationWantsResponse) {
	        //noinspection unchecked Kotlin compiler guarantees ReturnT to be Object.
	        return (HttpServiceMethod<ResponseT, ReturnT>)
	            new SuspendForResponse<>(
	                requestFactory,
	                callFactory,
	                responseConverter,
	                (CallAdapter<ResponseT, Call<ResponseT>>) callAdapter);
	      } else {
	        //noinspection unchecked Kotlin compiler guarantees ReturnT to be Object.
	        return (HttpServiceMethod<ResponseT, ReturnT>)
	            new SuspendForBody<>(
	                requestFactory,
	                callFactory,
	                responseConverter,
	                (CallAdapter<ResponseT, Call<ResponseT>>) callAdapter,
	                continuationBodyNullable);
	      }
	    } 
	  ```
- ## 1、创建callAdapter，从Retrofit build 时传入的callAdapter工厂里取的，
	- [[追溯CallAdapted构造函数，寻找CallAdapter]]
	- 作用：将okhttpCall 封装成 Retrofit 的 call
- ## 2、创建responseConverter也是
	- [[Converter获取流程]]
	- 作用：响应数据处理用