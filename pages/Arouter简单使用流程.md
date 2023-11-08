## 1、声明路由
collapsed:: true
	- 模块A使用@Arouter（path = “/order/mainActivity”） 声明路由，
- ## 2、使用了注解的每个模块编译期，通过APT生成路由表，都实现了一个接口
	- group实现
		- ```java
		  public interface IRouteGroup {
		      /**
		       * Fill the atlas with routes in group.
		       */
		      void loadInto(Map<String, RouteMeta> atlas);
		  }
		  ```
	- 为AMS自动注册路由表到内存 做准备
- ## 3、其他模块使用路由，Navigattion 去查找路由表，找到对应的类去跳转
  collapsed:: true
	- ```java
	  ARouter.getInstance().build(RouteHub.QRCode.QRCODE_SCAN_PATH).navigation(this)
	  ```