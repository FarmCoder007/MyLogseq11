- ## 一、什么是动态代理
  collapsed:: true
	- 首先，动态代理是代理模式的一种实现方式，代理模式除了动态代理还有 静态代理，只不过静态代理能够在编译时期确定类的执行对象，而动态代理只有在运行时才能够确定执行对象是谁。代理可以看作是对最终调用目标的一个封装，我们能够通过操作代理对象来调用目标类，这样就可以实现调用者和目标对象的解耦合。
	- 动态代理的应用场景有很多，最常见的就是 AOP 的实现、RPC 远程调用、Java 注解对象获取、日志框架、全局性异常处理、事务处理等。
	- 动态代理的实现有很多，但是 JDK 动态代理是很重要的一种
- ## 二、实现动态代理需要具备的因素
  collapsed:: true
	- 被代理的接口
	- 接口实现类
	- 通过方法匹配代理方法(InvocationHandler中的invoke)
- ## 三、动态代理的作用
  collapsed:: true
	- 1、方法插入代码：通过 接口中的 invoke 方法进行业务的调用和增强等处理，InvocationHandler是一个拦截器类
	- 2、api实现类的差异化，用代理类新实现替换旧实现
- ## 四、动态代理执行过程
	- ![image.png](../assets/image_1659671333414_0.png)
	- RD无法直接创建 “目标接口代理实现类对象”，通过创建Proxy.newInstance创建 “代理类对象”，调用invoke来访问目标对象的方法
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
  collapsed:: true
	- 一般组件化在api库定义 `api接口`，实现库定义接口 `实现类impl`。对外暴露的api会用 `接口包装类` 获取接口 `实现类` 去调用api，而不是直接去拿`实现类`去调用，后期底层换api，或者实现类，对外暴露的`接口包装类`的api不会变，上层业务也不需要改变。
	- 动态代理也可以在接口包装类中，拿到代理对象，替换原实现类对象，这样对外暴露的接口包装类，用的实际上是代理类
	- 接口api:
	  collapsed:: true
		- ```kotlin
		  /**
		   *  api接口 借助arouter 依赖注入 获取实例
		   */
		  public interface INetService {
		  
		      void doGet();
		  
		      void doPost();
		  
		      void uploadFile();
		  }
		  ```
	- 原实现类：
	  collapsed:: true
		- ```java
		  /**
		   *  实现类
		   */
		  @Route(path = RouterPath.TEST)
		  class NetServiceImpl : INetService {
		      override fun doGet() {
		          Log.i("MetaXDemo", "NetServiceImpl doGet")
		      }
		  
		      override fun doPost() {
		          Log.i("MetaXDemo", "NetServiceImpl doPost")
		      }
		  
		      override fun uploadFile() {
		          Log.i("MetaXDemo", "NetServiceImpl uploadFile")
		      }
		  }
		  ```
	- apt生成api接口包装类
	  collapsed:: true
		- ```java
		  /**
		   * APT 自动生成的工具类
		   * @author MetaX-Auto on 2022/08/04 19:23:00
		   */
		  public class NetWorkUtils {
		    private static INetService netService;
		  
		    private static boolean isInitProxy;
		  
		    public static synchronized INetService getNetService(Context context) {
		      if (netService != null) {
		        checkProxy(context);
		        return netService;
		      }
		      Object object = ARouter.getInstance().build("/demo1/netService").navigation();
		      if (object instanceof INetService) {
		        netService = (INetService) object;
		        checkProxy(context);
		      }
		      return netService;
		    }
		  
		    private static void checkProxy(Context context) {
		      if(MetaXCustomManager.hasCustomProxy("/demo1/netService") && !isInitProxy) {
		        // 拿到代理类对象  替换工具类实例
		        Object netServiceProxy = MetaXCustomManager.getProxy(context,"/demo1/netService",netService);
		        if (netServiceProxy != null) {
		          netService = (INetService) netServiceProxy;
		          isInitProxy = true;
		        }
		      }
		    }
		  
		    public static void doGet(Context context) {
		      INetService netService = getNetService(context);
		      if (netService != null) {
		        netService.doGet();
		      }
		    }
		  
		    public static void doPost(Context context) {
		      INetService netService = getNetService(context);
		      if (netService != null) {
		        netService.doPost();
		      }
		    }
		  
		    public static void uploadFile(Context context) {
		      INetService netService = getNetService(context);
		      if (netService != null) {
		        netService.uploadFile();
		      }
		    }
		  }
		  
		  ```
	- apt生成代理基类
	  collapsed:: true
		- ```java
		  /**
		   * APT 生成的代理基类，内部注册方法获取代理类实例
		   * 若要修改代理类实现，则继承该基类，重新 想要修改的方法
		   * @author MetaX-Auto on 2022/08/04 19:23:00
		   */
		  public class NetServiceBaseProxy implements INetService, IMetaXBaseProxy {
		    /**
		     *  原实现类对象
		     */
		    private INetService netService;
		  
		    public NetServiceBaseProxy() {
		    }
		  
		    /**
		     *  不重写该方法，则调用的还是原实现类对象的方法
		     */
		    @Override
		    public void doGet() {
		      if (netService != null) {
		        netService.doGet();
		      }
		    }
		  
		    @Override
		    public void doPost() {
		      if (netService != null) {
		        netService.doPost();
		      }
		    }
		  
		    @Override
		    public void uploadFile() {
		      if (netService != null) {
		        netService.uploadFile();
		      }
		    }
		  
		    /**
		     * 注册代理类
		     * @param context
		     * @param service 原实现类对象
		     * @return 返回代理类对象
		     */
		    @Override
		    public final Object register(@NonNull Context context, @NonNull Object service) {
		      // 原实现类对象
		      netService = (INetService)service;
		      /**
		       *  返回代理类对象
		       */
		      return ProxyUtils.callProxy(netService,new IMethodProxy() {
		        /**
		         *  调用代理方法时候  会调用此方法做分发
		         * @param who 代理类对象
		         * @param method 代理方法名
		         * @param args 对应方法参数
		         * @return 返回调用方法的返回值
		         * @throws Throwable 抛出异常
		         */
		        @Nullable
		        @Override
		        public Object call(@Nullable Object who, @Nullable Method method, @Nullable Object... args)
		            throws Throwable {
		          if(method == null) {
		            return null;
		          }
		          /**
		           * 当调用的doGet方法时，调用
		           */
		          if (method.getName().equals("doGet")) {
		            doGet();
		          }
		          if (method.getName().equals("doPost")) {
		            doPost();
		          }
		          if (method.getName().equals("uploadFile")) {
		            uploadFile();
		          }
		          return null;
		        }
		      });
		    }
		  }
		  
		  ```
	- 代理管理类
	  collapsed:: true
		- ```kotlin
		  /**
		   * Metax组件自定义api代理管理类
		   */
		  object MetaXCustomManager {
		  
		      /**
		       *  存储注册的<接口实现类路由地址，自定义代理类>
		       */
		      private val mApiMap = ConcurrentHashMap<String, Class<IMetaXBaseProxy>>()
		  
		      /**
		       * 提前把自定义代理类添加到map中
		       *
		       * 注册自定义的代理类，只传入class，用到的时候才会实例化，一般提前注册
		       */
		      @JvmStatic
		      fun <T : IMetaXBaseProxy> add(routePath: String, clazz: Class<T>): MetaXCustomManager {
		          if (mApiMap.contains(routePath)) {
		              throw IllegalArgumentException("route path:$routePath is already exist")
		          }
		          mApiMap[routePath] = clazz as Class<IMetaXBaseProxy>
		          return this
		      }
		  
		      /**
		       * 初始化代理类，调用register方法
		       */
		      @JvmStatic
		      fun getProxy(context: Context, routePath: String, service: Any): Any? {
		          //判断是否有自定义的代理
		          val clazz = mApiMap[routePath]
		          if (clazz == null) {
		              Log.e("MetaXApiCustomManager", "initProxy: $routePath doesn't exist")
		              return null
		          }
		          return try {
		              // 调用自定义代理类构造  创建CustomNetService 实例
		              clazz.getConstructor().newInstance().apply {
		                  // 调用CustomNetService 中的register方法
		                  register(context, service)
		              }
		          } catch (e: Exception) {
		              Log.e("MetaXApiCustomManager", "initProxy: ", e)
		              null
		          }
		      }
		  
		      /**
		       * 是否有自定义的代理类
		       */
		      @JvmStatic
		      fun hasCustomProxy(routePath: String): Boolean {
		          return mApiMap.containsKey(routePath)
		      }
		  
		  
		      /**
		       * 移除自定义的代理类
		       */
		      @JvmStatic
		      fun remove(routePath: String) {
		          if (hasCustomProxy(routePath)) {
		              mApiMap.remove(routePath)
		          }
		      }
		  
		      /**
		       * 清除所有代理类
		       */
		      @JvmStatic
		      fun clear() {
		          mApiMap.clear()
		      }
		  }
		  ```
	- 通用代理调度接口，代替直接使用InvocationHandler，统一管理 调用代理方法异常try
	  collapsed:: true
		- ```kotlin
		  interface IMethodProxy {
		      @Throws(Throwable::class)
		      fun call(who: Any?, method: Method?, vararg args: Any?): Any?
		  }
		  ```
	- 通用代理注册接口
	  collapsed:: true
		- ```kotlin
		  interface IMetaXBaseProxy {
		      /**
		       * 注册代理接口对象
		       * @param context
		       * @param service 原实现类
		       */
		      fun register(context: Context, service: Any): Any?
		  }
		  ```
	- 创建代理类方法
	  collapsed:: true
		- ```kotlin
		  object ProxyUtils {
		  
		  //     lambda形式
		  
		  //    @JvmStatic
		  //    fun <T> callProxy(obj: Any? , methodProxy: IMethodProxy): T? {
		  //        obj?.let {
		  //            val proxyInterfaces = MethodParameterUtils.getAllInterface(it.javaClass)
		  //            return Proxy.newProxyInstance(
		  //                    it.javaClass.classLoader , proxyInterfaces , InvocationHandler { proxy , method , args ->
		  //                try {
		  //                    return@InvocationHandler methodProxy.call(it , method , *args)
		  //                } catch (throwable: Throwable) {
		  //                    return@InvocationHandler null
		  //                }
		  //            }) as T
		  //        }
		  //        return null
		  //    }
		  
		      /**
		       * 参数1： 传入原实现类
		       * 参数2：代理方法接口 整体做异常捕获
		       *
		       * 返回值：返回创建的代理类实例
		       */
		      @JvmStatic
		      fun <T> callProxy(obj: Any? , methodProxy: IMethodProxy): T? {
		          obj?.let {
		              val proxyInterfaces = MethodParameterUtils.getAllInterface(it.javaClass)
		              /**
		               *  参数1：原实现类的 类加载器
		               *  参数2：原实现类的所有实现接口
		               *  参数3：InvocationHandler 子类对象，借助
		               *  返回值：创建的代理实例
		               */
		              return Proxy.newProxyInstance(it.javaClass.classLoader , proxyInterfaces , object :InvocationHandler{
		                      override fun invoke(proxy: Any?, method: Method?, args: Array<out Any>?): Any? {
		                          // 对包装的代理方法的调用，做整体的异常捕获
		                          return try {
		                              methodProxy.call(it , method , args)
		                          } catch (throwable: Throwable) {
		                              null
		                          }
		                      }
		                  }) as T
		          }
		          return null
		      }
		  }
		  ```
- ## 七、动态代理原理(反射)
	- 1、JavaDoc 告诉我们，InvocationHandler 是一个接口，实现这个接口的类就表示该类是一个代理实现类，也就是代理类
	- 2、动态代理的优势在于能够很方便的对代理类中方法进行集中处理，而不用修改每个被代理的方法。因为所有被代理的方法（真正执行的方法）都是通过在 InvocationHandler 中的 invoke 方法调用的。所以我们只需要对 invoke 方法进行集中处理
	- ## Proxy.newInstance 方法分析
	  collapsed:: true
		- ```java
		  /**
		   * @params loader 原实现类的类加载器
		   * @params interfaces 原实现类的实现的所有接口
		   * @params h InvocationHandler 对象
		   * 返回一个代理对象
		   */
		  public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces,
		                                            InvocationHandler h)
		          throws IllegalArgumentException
		      {
		          Objects.requireNonNull(h);
		          // 1、克隆接口   
		          final Class<?>[] intfs = interfaces.clone();
		          // Android-removed: SecurityManager calls
		          /*
		          final SecurityManager sm = System.getSecurityManager();
		          if (sm != null) {
		              checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
		          }
		          */
		  
		          /*
		           * 2、查找或生成指定的代理类class
		           */
		          Class<?> cl = getProxyClass0(loader, intfs);
		  
		          /*
		           * Invoke its constructor with the designated invocation handler.
		           */
		          try {
		              // Android-removed: SecurityManager / permission checks.
		              /*
		              if (sm != null) {
		                  checkNewProxyPermission(Reflection.getCallerClass(), cl);
		              }
		              */
		              // 3、 获取代理class的 构造器
		              final Constructor<?> cons = cl.getConstructor(constructorParams);
		              final InvocationHandler ih = h;
		              // 4、判断访问权限不是public  访问权限设置可访问
		              if (!Modifier.isPublic(cl.getModifiers())) {
		                  // BEGIN Android-removed: Excluded AccessController.doPrivileged call.
		                  /*
		                  AccessController.doPrivileged(new PrivilegedAction<Void>() {
		                      public Void run() {
		                          cons.setAccessible(true);
		                          return null;
		                      }
		                  });
		                  */
		  
		                  cons.setAccessible(true);
		                  // END Android-removed: Excluded AccessController.doPrivileged call.
		              }
		              // 5、构造器创建实例
		              return cons.newInstance(new Object[]{h});
		          } catch (IllegalAccessException|InstantiationException e) {
		              throw new InternalError(e.toString(), e);
		          } catch (InvocationTargetException e) {
		              Throwable t = e.getCause();
		              if (t instanceof RuntimeException) {
		                  throw (RuntimeException) t;
		              } else {
		                  throw new InternalError(t.toString(), t);
		              }
		          } catch (NoSuchMethodException e) {
		              throw new InternalError(e.toString(), e);
		          }
		      }
		  ```
		- newProxyInstsance 方法最重要的几个环节就是获得代理类、获得构造器，然后构造新实例
			- 获得构造器，然后构造新实例主要是反射
	- ## getProxyClass0() 获取代理类分析：
	  collapsed:: true
		- ```java
		  
		      /**
		       * a cache of proxy classes
		       */
		      private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
		          proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());
		  ```
		- ```java
		      private static Class<?> getProxyClass0(ClassLoader loader,
		                                             Class<?>... interfaces) {
		         // 原实现类实现接口的数量   接口数量是否大于 65535  
		         if (interfaces.length > 65535) {
		              throw new IllegalArgumentException("interface limit exceeded");
		          }
		  
		          // If the proxy class defined by the given loader implementing
		          // the given interfaces exists, this will simply return the cached copy;
		          // otherwise, it will create the proxy class via the ProxyClassFactory 
		          // 缓存代理类的proxyClassCache   
		        return proxyClassCache.get(loader, interfaces);
		      }
		  ```
		- 然后会直接从 proxyClassCache 中根据 loader 和 interfaces 获取代理对象实例。
		- 如果能够根据 loader 和 interfaces 找到代理对象，将会返回缓存中的对象副本；
		- 否则，它将通过 ProxyClassFactory 创建代理类。初始化的时候传入了ProxyClassFactory代理类创建工厂
		- proxyClassCache 是如何进行缓存的，
			- 只需要知道它的缓存时机就可以了：即在类加载的时候进行缓存。
	- ## ProxyClassFactory 创建代理类工厂
		- ```java
		  
		  //一个工厂函数，在给定类加载器和接口数组的情况下生成、定义和返回代理类。
		  private static final class ProxyClassFactory
		          implements BiFunction<ClassLoader, Class<?>[], Class<?>>
		      {
		          // 所有代理类名的前缀
		          private static final String proxyClassNamePrefix = "$Proxy";
		  
		          // 下一个用于生成唯一代理类名的数字
		          //这个属性表明 ProxyClassFactory 的后缀是使用 AtomicLong 生成的数字
		          private static final AtomicLong nextUniqueNumber = new AtomicLong();
		  
		          // 这个 apply 方法是一个根据接口和类加载器进行代理实例创建的工厂方法，下面是这段代码的核心。
		          @Override
		          public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {
		  
		              Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
		              for (Class<?> intf : interfaces) {
		                  /*
		                   * Verify that the class loader resolves the name of this
		                   * interface to the same Class object.
		                   */
		                  Class<?> interfaceClass = null;
		                  try {
		                      interfaceClass = Class.forName(intf.getName(), false, loader);
		                  } catch (ClassNotFoundException e) {
		                  }
		                  if (interfaceClass != intf) {
		                      throw new IllegalArgumentException(
		                          intf + " is not visible from class loader");
		                  }
		                  /*
		                   * Verify that the Class object actually represents an
		                   * interface.
		                   */
		                  if (!interfaceClass.isInterface()) {
		                      throw new IllegalArgumentException(
		                          interfaceClass.getName() + " is not an interface");
		                  }
		                  /*
		                   * Verify that this interface is not a duplicate.
		                   */
		                  if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
		                      throw new IllegalArgumentException(
		                          "repeated interface: " + interfaceClass.getName());
		                  }
		              }
		  
		              String proxyPkg = null;     // package to define proxy class in
		              int accessFlags = Modifier.PUBLIC | Modifier.FINAL;
		  
		              /*
		               * Record the package of a non-public proxy interface so that the
		               * proxy class will be defined in the same package.  Verify that
		               * all non-public proxy interfaces are in the same package.
		               */
		              for (Class<?> intf : interfaces) {
		                  int flags = intf.getModifiers();
		                  if (!Modifier.isPublic(flags)) {
		                      accessFlags = Modifier.FINAL;
		                      String name = intf.getName();
		                      int n = name.lastIndexOf('.');
		                      String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
		                      if (proxyPkg == null) {
		                          proxyPkg = pkg;
		                      } else if (!pkg.equals(proxyPkg)) {
		                          throw new IllegalArgumentException(
		                              "non-public interfaces from different packages");
		                      }
		                  }
		              }
		  
		              if (proxyPkg == null) {
		                  // if no non-public proxy interfaces, use the default package.
		                  proxyPkg = "";
		              }
		  
		              {
		                  // Android-changed: Generate the proxy directly instead of calling
		                  // through to ProxyGenerator.
		                  // 1、
		                  List<Method> methods = getMethods(interfaces);
		                  Collections.sort(methods, ORDER_BY_SIGNATURE_AND_SUBTYPE);
		                  validateReturnTypes(methods);
		                  List<Class<?>[]> exceptions = deduplicateAndGetExceptions(methods);
		  
		                  Method[] methodsArray = methods.toArray(new Method[methods.size()]);
		                  Class<?>[][] exceptionsArray = exceptions.toArray(new Class<?>[exceptions.size()][]);
		  
		                  /*
		                   * Choose a name for the proxy class to generate.
		                   */
		                  long num = nextUniqueNumber.getAndIncrement();
		                  String proxyName = proxyPkg + proxyClassNamePrefix + num;
		  
		                  return generateProxy(proxyName, interfaces, loader, methodsArray,
		                                       exceptionsArray);
		              }
		          }
		      }
		  ```