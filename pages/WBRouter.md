- 一、wbrouter依赖注入支持单例
	- 背景：@Router 注解 使用在接口服务获取实例时，一直是创建的新对象，
	- 需求：Router 添加新参数singleton=true，可选则的使用获取实例使用单例
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
			- 1、Route ：注解类添加可选参数 boolean singleton() default false; 是否使用单例
			- 2、RouteMeta：注解信息类
		-
	-