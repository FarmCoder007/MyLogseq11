public:: true

- ## 详细见 [[AMS启动流程-粗略]]
- # 前言、AMS是在SystemServer中被添加的，
- # 一、AMS对象的创建
	- ## 1、SystemServer.run()方法中
		- ### 1、创建SystemServiceManager(系统服务管理器)
		- ### 2、启动引导服务，核心服务，其他服务，AMS就是在引导服务中添加的
	- ##  2、在启动引导服务中，创建了AMS对象
		- ### 1、通过SystemServiceManager.startService，一系列调用，最终反射初始化[[#red]]==**AMS.Lifecycle对象**==
		- ### 2、Lifecycle的构造函数中初始化AMS对象
- # 二、AMS启动及相关配置
	- ## 3、SystemService.onStart()启动AMS
	- ## 4、通过setSystemServiceManager设置AMS的[[#red]]==**系统服务管理器**==
	- ## 5、通过setInstaller设置AMS的[[#red]]==**APP安装器**==
	- ## 6、初始化[[#red]]==**WMS**==(WindowsManagerService)、==**PMS**==(PowerManagerService)等和AMS相关的服务
	- ## 7、通过setSystemProcess将AMS、PMS、WMS等服务注册到ServiceManager中，以便其他组件通过SM获取 注册这些服务对应的Binder对象，进行IPC通信
	- ## 8、AMS.==**systemReady**==初始化完成
-