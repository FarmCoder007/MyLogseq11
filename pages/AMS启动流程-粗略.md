- ![image.png](../assets/image_1688881426102_0.png)
- # 一、[[#red]]==**AMS是在SystemServer中被添加的， SystemServer.run()方法中做了如下几件事**==
  collapsed:: true
	- ## 1、createSystemContext();初始化系统资源
	- ## 2、[[#red]]==**创建SystemServiceManager(管理启动的各种服务)**==
	- ## 3、启动各种服务，其中AMS是启动引导服务的方法中启动的
		- ```java
		  startBootstrapServices();
		  startCoreServices();
		  startOtherServices();
		  ```
- # 二、startBootstrapServices启动引导服务
	- ```java
	  mActivityManagerService = mSystemServiceManager.startService(
	  							ActivityManagerService.Lifecycle.class).getService();
	  ```
	- 1、通过SystemServiceManager.startService反射初始化ActivityManagerService.Lifecycle对象
	- 2、Lifecycle的构造函数中初始化AMS对象
	- # 三、ActivityManagerService构造中初始化时的工作
		- ## 1、创建Android的ui线程
		- ## 2、创建ActivityServices(管理Activity的)
		- ## 3、创建[[ActivityStackSupervisor]]对象，记录着Activity状态信息
		- ## 4、ClientLifecycleManager app Activity生命周期相关（api28以后）
	- # 四、startService调用到Lifecycle的onStart()会调用到AMS的start方法
		- 1、==移除所有的应用进程组==
		- 2、启动CpuTracker线程
		- 3、==启动电池统计服务==
		- 4、创建LocalService，并添加到LocalServices
- # 五、startBootstrapServices 其他工作
	- 1、mActivityManagerService.setInstaller  设置AMS的APP安装器
	- 2、mActivityManagerService.initPowerManagement 初始化AMS相关的PMS
	- 3、mActivityManagerService.setSystemProcess 设置SystemServer
- # 六、setSystemProcess 主要工作
	- 1、注册各种服务到ServiceManager中，以便其他组件通过SM获取这些服务对应的binder，进行进程间通信
	  collapsed:: true
		- activity  AMS
		- procstats  进程统计
		- meminfo 内存
		- gfxinfo 图像信息
		- dbinfo 数据库
		- cpuinfo CPU
		- permission  权限
		- processinfo 进程服务
		- usagestats 应用的使用情况
	- 2、性能统计相关的mSystemThread.installSystemApplicationInfo  用于创建性能统计的Profiler对象
	- 3、创建ProcessRecord对象
-
-
-