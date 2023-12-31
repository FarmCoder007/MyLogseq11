- 一、wbrouter依赖注入支持单例
	- 背景：@Router 注解 使用在接口服务获取实例时，一直是创建的新对象，
	- 需求：Router 添加新参数singleton=true，可选则的使用获取实例使用单例
	  collapsed:: true
		- ```
		  @Route(path=DemoRouters.LOCATION_SERVICE, singleton=true)
		  class LocationServiceImpl : ILocationService {
		      override fun startLocation(context: Context, callback: LocationCallback) {
		          val location = Location()
		          location.latitude = 31.225
		          location.longitude = 160.551
		          location.locationText = "北京市"
		          callback.onCallback(location)
		      }
		  
		      override fun stopLocation(context: Context) {
		          Toast.makeText(context, "停止定位成功", Toast.LENGTH_LONG).show()
		      }
		  }
		  ```
	- 方案：
		- 1、Annotation模块
		  collapsed:: true
			- 1、Route ：注解类添加可选参数 boolean singleton() default false; 是否使用单例
			- 2、RouteMeta：注解信息类，写入生成组文件
				- RouteProcessor 处理Route注释时，在生成组信息时，将Route中的singleton信息获取到，写入自动生成的组信息中，详见Process模块
				- 注：此时的addStatement中的占位符 只支持特定的几个详细见CodeBlock 头部注解
		- 2、Process模块：解析注解，将信息写入生成的group文件
		  collapsed:: true
			- ```
			         generatedGroup方法:          
			                   /*
			                   * atlas.put("/app/single_map_page", RouteMeta.build(RouteType.ACTIVITY, SinglePageMapActivity.class, "app", "/app/single_map_page", new java.util.HashMap<String, Integer>(){{put("extras", 3); put("params", 11); }}, -1, -2147483648));
			                   */
			                  loadIntoMethodBuilder.addStatement("atlas.put($S, $T.build($T.$L, $T.class, $S, $S, " + (StringUtils.isEmpty(mapBody) ? null : ("new java.util.HashMap<String, Integer>(){{" + mapBody + "}}")) + ", $S, $L,$L))",
			                          routeMeta.getPath(),//路由路径
			                          ClassName.get(RouteMeta.class),//RouteMeta
			                          ClassName.get(RouteType.class),//RouteType
			                          routeMeta.getRouteType(),//type
			                          className,//target className ,such as MainActivity.class
			                          routeMeta.getGroup(),//route group
			                          routeMeta.getPath(),//route path
			                          routeMeta.getTargetMethodName(),
			                          routeMeta.getExtraFlags(),//extension parameters
			                          routeMeta.isSingleton()// 是否要生成单例
			                  );
			  ```
		- 3、demo模块
			- 1、服务接口
			  collapsed:: true
				- ```
				  public interface HelloService {
				      void sayHello(String name);
				  }
				  ```
			- 2、接口实现类，希望依赖注入获取的是单例，使用注解singleton
			  collapsed:: true
				- ```
				  @Route(value = "/yourservicegroupname/hello",singleton = true)
				  public class HelloServiceImpl implements HelloService {
				      Context mContext;
				  
				      @Override
				      public void sayHello(String name) {
				          Toast.makeText(mContext, "Hello " + name, Toast.LENGTH_SHORT).show();
				      }
				  
				      /**
				       * Do your init work in this method, it well be call when processor has been load.
				       *
				       * @param context ctx
				       */
				      public void init(Context context) {
				          mContext = context;
				          Log.e("testService", HelloService.class.getName() + " has init.");
				      }
				  }
				  ```
			- 3、build中自动生成的组文件：是processor编译时自动生成的
			  collapsed:: true
				- ```
				  public class WBRouter$$Group$$sample$$yourservicegroupname implements IRouteGroup {
				    @Override
				    public void loadInto(Map<String, RouteMeta> atlas) {
				      atlas.put("/yourservicegroupname/hello", RouteMeta.build(RouteType.CUSTOMIZATION, HelloServiceImpl.class, "yourservicegroupname", "/yourservicegroupname/hello", null, null, 0,true));
				      atlas.put("/yourservicegroupname/hel", RouteMeta.build(RouteType.CUSTOMIZATION, com.wuba.wbrouter.sample.router.HelloServiceImpl.class, "yourservicegroupname", "/yourservicegroupname/hel", null, null, 0,true));
				    }
				  }
				  ```
		- 4、core模块，处理依赖注入逻辑
		  collapsed:: true
			- 1、WBRouterCore，当调用navigation时，获取group中的路由信息，在自定义类型RouteType.CUSTOMIZATION 添加实例缓存支持
			  collapsed:: true
				-
				- ```
				  RouteType.CUSTOMIZATION -> {
				                  //自定义路由类型
				                  val metaClass: Class<*> = routePacket.targetClass
				                  try {
				                      return if(routePacket.isSingleton){
				                          // 使用单例先从缓存取
				                          var instance = Warehouse.serviceInstance[metaClass]
				                          if(instance == null){
				                              instance = getInstance(metaClass,routePacket,context)
				                              Warehouse.serviceInstance[metaClass] = instance
				                          }
				                          instance
				                      } else {
				                          // 不使用单例，每次创建对象
				                          getInstance(metaClass,routePacket,context)
				                      }
				                  } catch (ex: Exception) {
				                      sendError(ex)
				                  }
				                  return null
				              }
				              
				              
				     // 获取实例         
				     private fun getInstance(metaClass: Class<*>,routePacket: RoutePacket,context: Context?):Any?{
				          val instance = metaClass.getConstructor().newInstance()
				          routePacket.targetMethodName?.let {
				              val method = metaClass.getMethod(routePacket.targetMethodName, Context::class.java, RoutePacket::class.java);
				              method.invoke(instance, context, routePacket)
				          }
				          return instance
				      }
				  ```
				-
				-
				-
				-
			- 2、LogisticsCenter，将自动生成的路由信息加载到内存map中，手动把加的字段也导入
			  collapsed:: true
				- ```
				      private fun importFormMeta(routePacket: RoutePacket, routeMeta: RouteMeta) {
				          routePacket.targetClass = routeMeta.targetClass
				          routePacket.setRouteType(routeMeta.getRouteType())
				          //postcard.setRawType(routeMeta.rawType)
				          routePacket.extraFlags = routeMeta.extraFlags
				          routePacket.addFlags(routeMeta.extraFlags)
				          routePacket.targetMethodName = routeMeta.targetMethodName
				          routePacket.setSingleton(routeMeta.isSingleton)
				          when (routeMeta.routeType) {
				              RouteType.FRAGMENT, RouteType.CUSTOMIZATION -> routePacket.isGreenChannel = true
				              else -> routePacket.isGreenChannel = false
				          }
				      }
				  ```
			- 3、Warehouse 全局缓存
				- ```
				  class Warehouse {
				      // Cache route and metas
				      volatile static Map<String, Class<? extends IRouteGroup>> groupsIndex = new UniqueKeyHashMap<>();
				      volatile static Map<String, RouteMeta> routes = new UniqueKeyHashMap<>();
				  
				      static Map<Class, Object> serviceInstance = new UniqueKeyHashMap<>();
				      // 所有已注解定义的拦截器
				      static Map<String, InterceptorMeta> allInterceptors = new UniqueKeyLinkedHashMap<>("More than one interceptors use same priority [%s]");
				      // 白名单中拦截器，按优先级存放
				      static List<InterceptorMeta> whiteInterceptors = new ArrayList<>();
				  
				      static void clear() {
				          routes.clear();
				          groupsIndex.clear();
				          whiteInterceptors.clear();
				          allInterceptors.clear();
				          serviceInstance.clear();
				      }
				  }
				  ```
		-
	-