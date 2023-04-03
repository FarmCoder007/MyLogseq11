- ## 1、结合程序的编译和执行过程，说明怎样添加注解、添加什么样的注解可以实现自动生成代码、编译器怎样根据注解自动生成代码。
  collapsed:: true
	- ![image.png](../assets/image_1680502329046_0.png){:height 304, :width 746}
	- 具体生成代码工具怎样生成代码：
	  collapsed:: true
		- ![image.png](../assets/image_1680502345900_0.png)
	-
- ## 2、结合程序的编译和执行过程，说明在哪里添加注解、添加什么样的注解可以实现自动生成代码，且即使换内部依赖注入框架，也不影响上层API的调用。（即结合程序的编译和执行过程，说明接口包装工具类的作用、生成方法、使用方法）
	- 前提条件，旧方案和本次优化方法涉及地方
	- 定位库举例：
		- 1、定义定位接口：
			- ```kotlin
			  @MetaXApi(router = DemoRouters.LOCATION_SERVICE)
			  interface ILocationService {
			      fun startLocation(context: Context, callback: LocationCallback)
			      fun stopLocation(context: Context)
			  }
			  ```
		- 2、接口实现类：
		  collapsed:: true
			- ```kotlin
			  // MetaXRoute为路由框架的注解，不是本方法添加的，但是路由通用框架都是这样定义的
			  // 定义的路由标记，怎么在程序中找到他
			  @MetaXRoute(DemoRouters.LOCATION_SERVICE) 
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
	- 分享库、或者其他业务库使用
		- 旧方式1、直接通过路由框架获取 定位实现类实例，不进行包装，直接调用，后期换路由框架、或者换定位具体实现，需要修改成本较大
		- ```java
		  MetaXRouteCore这个是路由框架  ApiRouter 这个是路由标记  LocationServiceImpl  通过路由创建的对象
		  // 1、获取实例
		  LocationServiceImpl  impl =  MetaXRouteCore.navigation(ApiRouter)
		  // 2、调用方法
		  impl .startLocation()
		  impl .stopLocation()
		  ```
		- 缺点