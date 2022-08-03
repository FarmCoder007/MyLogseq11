- ## 一、object
	- ## 1、用于匿名内部类声明
		- ```kotlin
		  //  Proxy 创建代理类的方法
		  public static Object newProxyInstance(ClassLoader loader,
		                                          Class<?>[] interfaces,
		                                           InvocationHandler h)
		  // InvocationHandler匿名内部类传入
		  Proxy.newProxyInstance(classLoader , proxyInterfaces , object :InvocationHandler{
		                      override fun invoke(proxy: Any?, method: Method?, args: Array<out Any>?): Any {
		                          TODO("Not yet implemented")
		                      }
		                  })
		  ```
	- ## 2、用于半生对象声明
		-
	- ## 3、用于单例类声明
- ## 二、return
-