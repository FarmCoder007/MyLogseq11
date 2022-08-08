- ## 一、什么是动态代理
	- 首先，动态代理是代理模式的一种实现方式，代理模式除了动态代理还有 静态代理，只不过静态代理能够在编译时期确定类的执行对象，而动态代理只有在运行时才能够确定执行对象是谁。代理可以看作是对最终调用目标的一个封装，我们能够通过操作代理对象来调用目标类，这样就可以实现调用者和目标对象的解耦合。
	- 动态代理的应用场景有很多，最常见的就是 AOP 的实现、RPC 远程调用、Java 注解对象获取、日志框架、全局性异常处理、事务处理等。
	- 动态代理的实现有很多，但是 JDK 动态代理是很重要的一种
- ## 二、实现动态代理需要具备的因素
	- 被代理的接口
	- 接口实现类
	- 通过方法匹配代理方法(InvocationHandler中的invoke)
- ## 三、动态代理的作用
	- 1、方法插入代码：通过 接口中的 invoke 方法进行业务的调用和增强等处理，InvocationHandler是一个拦截器类
	- 2、api实现类的差异化，用代理类新实现替换旧实现
- ## 四、动态代理执行过程
	- ![image.png](../assets/image_1659671333414_0.png)
	- RD无法直接访问 “目标接口实现类对象”，通过创建 “代理类对象”，调用invoke来访问目标对象的方法
	- 在 JDK 动态代理中，实现了 InvocationHandler 的类可以看作是 代理类
- ## 五、相关类
  collapsed:: true
	- ### 5-1、InvocationHandler接口 (实现该接口作为代理类)
		- ```java
		  public interface InvocationHandler {
		     /**
		      * Object proxy: 动态代理对象,即调用该方法的代理实例
		      * Method method：表示最终要执行的方法，method.invoke 用于执行被代理的方法，也就是真正的目标方法（对应于在代理实例上调用的接口方法的方法实例）
		      * Object[] args：这个参数就是向目标方法传递的参数。
		      *
		      * 返回值Object：即代理方法的返回值，如果接口方法的声明返回类型是基元类型，则该方法返回的值必须是相应基元包装类的实例
		      */
		      public Object invoke(Object proxy, Method method, Object[] args)
		          throws Throwable;
		  }
		  ```
		- InvocationHandler接口对象是传入到Proxy.newProxyInstance的代理调度器，newProxyInstance获取到代理对象执行代理方法时，会分派到的该调度器的invoke方法
			- 调用newProxyInstance
				- 1、可以传入InvocationHandler实现类对象，重写invoke方法，
				- 2、传入InvocationHandler的匿名内部类，新建与InvocationHandler同api的自定义接口，在InvocationHandler的匿名内部类对象的invoke方法中，调用自定义的同名方法->见包装实现
	- ### 5-2、Proxy.newProxyInstance(创建代理类对象)
		- ```java
		   /**
		    *  1-2 参数与被代理类有关
		    *  ClassLoader loader:用于定义代理类的类加载器 一般使用原实现类类加载器 objImpl.javaClass.classLoader 
		    *  Class<?>[] interfaces:代理类要实现的接口列表  [可以获取原实现类的 所有父接口传入]
		    *  
		    *  InvocationHandler h：将方法调用分派到的该调度器，也就是调方法的时候，会调度到该接口的invoke方法
		    *  
		    *  返回值：返回指定接口的代理类实例
		    */
		  public static Object newProxyInstance(ClassLoader loader,
		                                            Class<?>[] interfaces,
		                                            InvocationHandler h){
		     
		   }
		  ```
- ## 六、封装使用
	- 一般组件化在api库定义 `api接口`，实现库定义接口 `实现类impl`。对外暴露的api会用 `接口包装类` 获取接口 `实现类` 去调用api，而不是直接去拿`实现类`去调用，后期底层换api，或者实现类，对外暴露的`接口包装类`的api不会变，上层业务也不需要改变。
	- 动态代理也可以在接口包装类中，拿到代理对象，替换原实现类对象，这样对外暴露的接口包装类，用的实际上是代理类
	- 接口api:
		- ```kotlin
		  ```
	- 原实现类：
	- apt生成api接口包装类
	- apt生成代理基类
	- 代理管理类
	- 通用代理调度接口，代替直接使用InvocationHandler，统一管理
	- 通用代理注册接口
	- 创建代理类方法
- ## 七、动态代理原理(反射)
	-
-