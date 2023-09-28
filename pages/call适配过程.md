- retrofit.create
	- ```java
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
- 详见 详细见[[create 源码及过程 2.9.0]]
- ### 1、每当有代理类对象进行网络请求，会回调create的动态代理中invoke方法
- ### 2、通过loadServiceMethod根据每个Method创建ServiceMethod实例调用其invoke方法。
- ### 3、实际调用ServiceMethod子类 CallAdapted的Adapt方法 包装okhttpCall返回一个ExecutorCallBackCall(可进行线程切换 切换到主线程)
-
- ### 具体call的适配是：
	- 1、Retrofit 将Okhttp的 realCall 封装到Retrofit中的OkhttpCall的类中。
	- 2、在创建ServiceMethod（默认是CallAdapted）时创建OkhttpCall对象，传给Adapted。CallAdapter进行包装转换 返回一个ExecutorCallBackCall。可以进行线程切换，切换到主线程的call
	- 3、CallAdapter是在Retrofit初始化的时候 build中添加的默认的CallAdapter
	- 4、CallAdapter.adapt 方法就是 把传入的okhttpCall 封装成ExecutorCallBackCall并返回