- # 背景
  collapsed:: true
	- APP在启动时通常可以带一些参数，Android系统会在Application初始化后，紧接着根据这些参数创建相应的页面Activity，同时会把这些参数传递给页面，然后完成程序到页面的启动。从系统提供的能力看，启动参数只有在启动页创建后通过页面Intent才可以获取，Application的初始化是在启动页创建之前完成的，它的初始化方法中系统并没有给其提供启动参数。
- # 问题
  collapsed:: true
	- 一些在Application初始化依赖启动参数的逻辑无法在初始化执行，很多都移到了启动页Activity的初始化方法中。比如：一般App会在LaunchActivity或HomeActivity中解析外部调起的协议数据再做分发。