title:: loadServiceMethod，2.9.0版本

- title:: loadServiceMethod，2.9.0版本
  ```java
  ServiceMethod<Object, Object> serviceMethod =
                                    (ServiceMethod<Object, Object>) loadServiceMethod(method);
  ```
- # loadServiceMethod描述
	- [[#red]]==**根据method 生成一个ServiceMethod对象**==（内部会将生成的ServiceMethod放入在缓存中，如果已经生成过则直接从缓存中获取）
- # 源码流程
	- ```java
	    ServiceMethod<?> loadServiceMethod(Method method) {
	      ServiceMethod<?> result = serviceMethodCache.get(method);
	      if (result != null) return result;
	  
	      synchronized (serviceMethodCache) {
	        result = serviceMethodCache.get(method);
	        if (result == null) {
	          result = ServiceMethod.parseAnnotations(this, method);
	          serviceMethodCache.put(method, result);
	        }
	      }
	      return result;
	    }
	  ```
	- ## 1、loadServiceMethod首先会从缓存中获取ServiceMethod对象，有就返回，没有则创建，然后添加到缓存里
	- ## 2、ServiceMethod.parseAnnotations去创建
		- ServiceMethod.java
			- ```java
			    static <T> ServiceMethod<T> parseAnnotations(Retrofit retrofit, Method method) {
			      // 1、通过 RequestFactory 解析方法上的注解
			      RequestFactory requestFactory = RequestFactory.parseAnnotations(retrofit, method);
			  
			      Type returnType = method.getGenericReturnType();
			      if (Utils.hasUnresolvableType(returnType)) {
			        throw methodError(
			            method,
			            "Method return type must not include a type variable or wildcard: %s",
			            returnType);
			      }
			      if (returnType == void.class) {
			        throw methodError(method, "Service methods cannot return void.");
			      }
			  
			      return HttpServiceMethod.parseAnnotations(retrofit, method, requestFactory);
			    }
			  ```
		- ### 2-1、[[RequestFactory初始化解析接口方法注解]]
			- > 对我们写的网络请求接口里发方法 做分析 记录   分析 所有的注解  方法   然后分析出完整的拼装方案存在 RequestFactory这里
		- ### 2-2、HttpServiceMethod.parseAnnotations去获取ServiceMethod,见下文
	- ## 3、内部调用的HttpServiceMethod的方法
		- HttpServiceMethod.java 继承ServiceMethod
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
		- 最下边判断非kotlin挂起函数就创建 CallAdapted
		- [[创建ServiceMethod的子类CallAdapted时具体干了什么]]
	- ## 4、[[CallAdapted]] 继承 HttpServiceMethod
	- ## **[[#red]]==[[#red]]阶段总结：返回的是loadServiceMethod返回的是ServiceMethod子类CallAdapted==**
	-
- # 细节
	- 每一个method 都有一个自己的ServiceMethod，这就意味着ServiceMethod是属于函数的，而不是类的。也就是我们定义的网络访问接口类，在接口类里面的每一个函数都会在反射阶段形成自己的serviceMethod。
	  id:: 64b8ade8-7ff7-4c40-86ba-139c9570818c
- ## [[serviceMethod是什么？]]