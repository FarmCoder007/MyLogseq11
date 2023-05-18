- # 简介
	- 目前大多数项目都采用的Retrofit作为自己的网络框架，而自己对于该框架的掌握程度还停留在CV阶段。是时候搞一波源码分析了！
	- Retrofit最让大家称道的是他可以让请求兼容rxJava，也可以轻松的将网络请求结果转换成自己想要的实体。那它究竟是怎么实现的呢？下来我们就扒拉下他的源码看看他是怎么实现这么牛逼的功能的。
- # 使用示例
	- 老样子，我们从一个简单的使用例子着手，逐步深入往里看。
	  collapsed:: true
		- ```
		  OkHttpClient.Builder httpClient = new OkHttpClient.Builder();
		          Retrofit retrofit = new Retrofit.Builder()
		                  .addConverterFactory(GsonConverterFactory.create(new GsonBuilder().create()))
		                  .addCallAdapterFactory(RxJava2CallAdapterFactory.createAsync())
		                  .baseUrl("https://api.douban.com/v2/").client(httpClient.build())
		                  .build();
		          BookService service = retrofit.create(BookService.class);
		  ```
	- 首先通过builder传入ConverterFactory，传入CallAdapterFactory。
	- 然后将OkHttpClient对象也塞了进去。
	- 最后调用create方法传入请求服务的接口，就可以构造出一个请求服务的实例对象。
- # 构造
  collapsed:: true
	- 我们看下create方法的源码。这里判断如果一个普通类里的方法就直接反射执行这个方法。
	- 如果是默认方法就就反射调用，不是的话就去加载ServiceMethod，然后去invoke。关于这段我们后边详述。
	  collapsed:: true
		- ```
		  public <T> T create(final Class<T> service) {
		      validateServiceInterface(service);
		      return (T)
		          Proxy.newProxyInstance(
		              service.getClassLoader(),
		              new Class<?>[] {service},
		              new InvocationHandler() {
		                private final Platform platform = Platform.get();
		                private final Object[] emptyArgs = new Object[0];
		  
		                @Override
		                public @Nullable Object invoke(Object proxy, Method method, @Nullable Object[] args)
		                    throws Throwable {
		                  // If the method is a method from Object then defer to normal invocation.
		                  if (method.getDeclaringClass() == Object.class) {
		                    return method.invoke(this, args);
		                  }
		                  args = args != null ? args : emptyArgs;
		                  return platform.isDefaultMethod(method)
		                      ? platform.invokeDefaultMethod(method, service, proxy, args)
		                      : loadServiceMethod(method).invoke(args);
		                }
		              });
		    }
		  ```
	- 这里第一个知识点来了，看下platform.isDefaultMethod(method)这个方法，method.isDefault()是干啥的，什么是默认方法。
	  collapsed:: true
		- ```
		  @IgnoreJRERequirement // Only called on API 24+.
		   boolean isDefaultMethod(Method method) {
		      return hasJava8Types && method.isDefault();
		    }
		  ```
	- 在 java 8 之前，接口与其实现类之间的 耦合度 太高了（tightly coupled），当需要为一个接口添加方法时，所有的实现类都必须随之修改。默认方法解决了这个问题，它可以为接口添加新的方法，而不会破坏已有的接口的实现,比如这样：
	  collapsed:: true
		- ```
		  interface InterfaceA {
		      default void foo() {
		          System.out.println("InterfaceA foo");
		      }
		  }
		  ```
	- 这么写之后，那么实现这个接口的类也不必非要实现foo这个方法。
	- 第二个知识点@IgnoreJRERequirement有啥干啥的？
	- 这个目前理解还是模棱两可，目前我的理解是这样的：有个方法在java6和java8都有，但是他们内部实现细节可能跟java环境不一样有各自区别。在真正编译的时候虚拟机会校验这个方法的签名。如果不满足环境要求就报错。加上这个注解就会跳过这个检查。[参考文档](https://hexilee.me/2018/09/27/animal-sniffer/)
	- 我们接着回到正题，对于我们安卓平台这个最后返回的是false。false走loadServiceMethod(method).invoke(args)逻辑。
	- 我们重点看下loadServiceMethod(method).invoke(args);这段代码实现。内部会先去缓存map里根据方法去查找map，如果查到就直接返回，否则调用ServiceMethod.parseAnnotations(this, method);进行解析。
		- ```
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
	- 我们继续往下跟就能发现解析的地方。
	  collapsed:: true
		- ```
		  abstract class ServiceMethod<T> {
		    static <T> ServiceMethod<T> parseAnnotations(Retrofit retrofit, Method method) {
		      RequestFactory requestFactory = RequestFactory.parseAnnotations(retrofit, method);
		      
		     ...
		     
		      return HttpServiceMethod.parseAnnotations(retrofit, method, requestFactory);
		    }
		  ```
	- 这块我们可以看到有两个地方做解析，一个是RequestFactory，一个是HttpServiceMethod。
	- 先看RequestFactory解析部分：
	  collapsed:: true
		- ```
		  private void parseMethodAnnotation(Annotation annotation) {
		        if (annotation instanceof DELETE) {
		          parseHttpMethodAndPath("DELETE", ((DELETE) annotation).value(), false);
		        } else if (annotation instanceof GET) {
		          parseHttpMethodAndPath("GET", ((GET) annotation).value(), false);
		        } else if (annotation instanceof HEAD) {
		          parseHttpMethodAndPath("HEAD", ((HEAD) annotation).value(), false);
		        } else if (annotation instanceof PATCH) {
		          parseHttpMethodAndPath("PATCH", ((PATCH) annotation).value(), true);
		        } else if (annotation instanceof POST) {
		          parseHttpMethodAndPath("POST", ((POST) annotation).value(), true);
		        } else if (annotation instanceof PUT) {
		          parseHttpMethodAndPath("PUT", ((PUT) annotation).value(), true);
		        } else if (annotation instanceof OPTIONS) {
		          parseHttpMethodAndPath("OPTIONS", ((OPTIONS) annotation).value(), false);
		        } else if (annotation instanceof HTTP) {
		          HTTP http = (HTTP) annotation;
		          parseHttpMethodAndPath(http.method(), http.path(), http.hasBody());
		        } else if (annotation instanceof retrofit2.http.Headers) {
		          String[] headersToParse = ((retrofit2.http.Headers) annotation).value();
		          if (headersToParse.length == 0) {
		            throw methodError(method, "@Headers annotation is empty.");
		          }
		          headers = parseHeaders(headersToParse);
		        } else if (annotation instanceof Multipart) {
		          if (isFormEncoded) {
		            throw methodError(method, "Only one encoding annotation is allowed.");
		          }
		          isMultipart = true;
		        } else if (annotation instanceof FormUrlEncoded) {
		          if (isMultipart) {
		            throw methodError(method, "Only one encoding annotation is allowed.");
		          }
		          isFormEncoded = true;
		        }
		      }
		  ```
	- 可以看到这部分就是把我们请求服务上的注解里的参数解析出来放到了RequestFactory.Builder里。
	- 我们在看HttpServiceMethod解析部分。
	  collapsed:: true
		- ```
		  /**
		     * Inspects the annotations on an interface method to construct a reusable service method that
		     * speaks HTTP. This requires potentially-expensive reflection so it is best to build each service
		     * method only once and reuse it.
		     */
		    static <ResponseT, ReturnT> HttpServiceMethod<ResponseT, ReturnT> parseAnnotations(
		        Retrofit retrofit, Method method, RequestFactory requestFactory) {
		      ...
		  
		      CallAdapter<ResponseT, ReturnT> callAdapter =
		          createCallAdapter(retrofit, method, adapterType, annotations);
		      ...
		  
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
	- 这里核心东西就来了。首先他根据外边传的callAdapterFactories创建了callAdapter，然后根据外边传的converterFactories创建了responseConverter对象。最终返回了HttpServiceMethod对象。
	- callAdapter是对网络请求事件的一次包装，他的作用就是将我们的这个请求封装成一个指定类型的东西，例如如果是rx实现那么就让他变成一个可观察的事件，由外部监听他。
	- responseConverter很好理解，无非就是之后网络请求结果回来之后，通过这个转换器将数据流转换成对应的实体数据bean。这个转换逻辑是外部实现，retrofit只调用这个能力并不关心具体的实现。
- # CallAdapter
  collapsed:: true
	- 我们接着往下瞅。上边create方法的动态代理处理类里，得到上边的HttpServiceMethod对象后会调用他的invoke方法。
	  collapsed:: true
		- ```
		   @Override
		    final @Nullable ReturnT invoke(Object[] args) {
		      Call<ResponseT> call = new OkHttpCall<>(requestFactory, args, callFactory, responseConverter);
		      return adapt(call, args);
		    }
		  ```
	- 首先创建了一个OkHttpCall对象，然后调用adapt抽象方法。我们具体找一个简单的实现类看看。
	  collapsed:: true
		- ```
		   @Override
		      protected ReturnT adapt(Call<ResponseT> call, Object[] args) {
		        return callAdapter.adapt(call);
		      }
		  ```
	- CallAdapter 是个接口我们继续找实现类。我们就找这个RxJavaCallAdapter这个经典的适配器来瞅瞅。
	  collapsed:: true
		- ```
		  @Override
		    public Object adapt(Call<R> call) {
		      OnSubscribe<Response<R>> callFunc =
		          isAsync ? new CallEnqueueOnSubscribe<>(call) : new CallExecuteOnSubscribe<>(call);
		  
		      OnSubscribe<?> func;
		      if (isResult) {
		        func = new ResultOnSubscribe<>(callFunc);
		      } else if (isBody) {
		        func = new BodyOnSubscribe<>(callFunc);
		      } else {
		        func = callFunc;
		      }
		      Observable<?> observable = Observable.create(func);
		  
		      if (scheduler != null) {
		        observable = observable.subscribeOn(scheduler);
		      }
		  
		      if (isSingle) {
		        return observable.toSingle();
		      }
		      if (isCompletable) {
		        return observable.toCompletable();
		      }
		      return observable;
		    }
		  
		  ```
	- 嗯，看到这里我们就搞明白了CallAdapter的作用。retrofit是通过这个适配器将我们的请求操作转换成一个observable对象，这样其他地方就可以进行订阅这个可监听事件，如此就完成了rxjava得适配器的转换工作。
- # responseConverter
	- 上边我们讲了CallAdapter这个东西可以将我们的请求转换成observable，这样我们就可以用rxjava来进行订阅这个事件。
	- 那我们继续顺着CallAdapter的代码继续往下看。
		- ```
		  OnSubscribe<Response<R>> callFunc =
		          isAsync ? new CallEnqueueOnSubscribe<>(call) : new CallExecuteOnSubscribe<>(call);
		  ```
	- 这里我们瞅见有个判断，是否是异步的。如果是异步构造的是CallEnqueueOnSubscribe这个东西，否则是CallExecuteOnSubscribe。我们在继续看他们各自的源码
		- ```
		  CallEnqueueOnSubscribe 源码
		  
		  @Override
		    public void call(Subscriber<? super Response<T>> subscriber) {
		      // Since Call is a one-shot type, clone it for each new subscriber.
		      Call<T> call = originalCall.clone();
		      final CallArbiter<T> arbiter = new CallArbiter<>(call, subscriber);
		      subscriber.add(arbiter);
		      subscriber.setProducer(arbiter);
		  
		      call.enqueue(
		          new Callback<T>() {
		            @Override
		            public void onResponse(Call<T> call, Response<T> response) {
		              arbiter.emitResponse(response);
		            }
		  
		            @Override
		            public void onFailure(Call<T> call, Throwable t) {
		              Exceptions.throwIfFatal(t);
		              arbiter.emitError(t);
		            }
		          });
		    }
		  ```
		- ```
		  CallExecuteOnSubscribe 源码
		  
		  @Override
		    public void call(Subscriber<? super Response<T>> subscriber) {
		      // Since Call is a one-shot type, clone it for each new subscriber.
		      Call<T> call = originalCall.clone();
		      CallArbiter<T> arbiter = new CallArbiter<>(call, subscriber);
		      subscriber.add(arbiter);
		      subscriber.setProducer(arbiter);
		  
		      Response<T> response;
		      try {
		        response = call.execute();
		      } catch (Throwable t) {
		        Exceptions.throwIfFatal(t);
		        arbiter.emitError(t);
		        return;
		      }
		      arbiter.emitResponse(response);
		    }
		  ```
	- 他们的区别其实就是调用Call的方法不一样，而这个Call最常用的子类其实就是我们外边看到的OkHttpCall这个东西。enqueue就是OKhttp的异步请求通过接口监听进行回调。而execute是同步请求，最终将结果emitResponse出去。
	- 我们在继续顺着这个线往下看就能看到数据解析的地方。
	  collapsed:: true
		- ```
		  Response<T> parseResponse(okhttp3.Response rawResponse) throws IOException {
		      ResponseBody rawBody = rawResponse.body();
		  
		      // Remove the body\'s source (the only stateful object) so we can pass the response along.
		      rawResponse =
		          rawResponse
		              .newBuilder()
		              .body(new NoContentResponseBody(rawBody.contentType(), rawBody.contentLength()))
		              .build();
		  
		      int code = rawResponse.code();
		      if (code < 200 || code >= 300) {
		        try {
		          // Buffer the entire body to avoid future I/O.
		          ResponseBody bufferedBody = Utils.buffer(rawBody);
		          return Response.error(bufferedBody, rawResponse);
		        } finally {
		          rawBody.close();
		        }
		      }
		  
		      if (code == 204 || code == 205) {
		        rawBody.close();
		        return Response.success(null, rawResponse);
		      }
		  
		      ExceptionCatchingResponseBody catchingBody = new ExceptionCatchingResponseBody(rawBody);
		      try {
		        T body = responseConverter.convert(catchingBody);
		        return Response.success(body, rawResponse);
		      } catch (RuntimeException e) {
		        // If the underlying source threw an exception, propagate that rather than indicating it was
		        // a runtime exception.
		        catchingBody.throwIfCaught();
		        throw e;
		      }
		    }
		  ```
	- 这块逻辑很清晰，如果请求结果小于200或者大于等于300就返回异常，如果204或者205就直接返回成功不做解析。如果这些都不是那么就用到了我们的responseConverter进行转换强转。
	-
-
- 参考：
	- https://hexilee.me/2018/09/27/animal-sniffer/