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
		- ```
		  ```
- 参考：
	- https://hexilee.me/2018/09/27/animal-sniffer/