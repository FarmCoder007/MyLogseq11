- [AMS.pptx](../assets/AMS_1688714848608_0.pptx)
- # 一、简介
	- ActivityManagerService是Android系统中一个特别重要的系统服务，也是我们上层APP打交道最多的系
	- 统服务之一。ActivityManagerService（以下简称AMS） 主要负责四大组件的启动、切换、调度以及应
	- 用进程的管理和调度工作。所有的APP应用都需要 与AMS打交道
- # 二、Activity Manager的组成
  collapsed:: true
	- 1.服务代理：
		- 由ActivityManagerProxy实现，用于与Server端提供的系统服务进行进程间通信
	- 2.服务中枢：
		- ActivityManagerNative继承自Binder并实现IActivityManager，它提供了服务接口和Binder接口的相互转化功能，并在内部存储服务代理对像，并提供了getDefault方法返回服务代理
	- 3.Client：
		- 由ActivityManager封装一部分服务接口供Client调用。ActivityManager内部通过调用ActivityManagerNative的getDefault方法，可以得到一个ActivityManagerProxy对像的引用，进而通过该代理对像调用远程服务的方法
	- 4.Server:
		- 由ActivityManagerService实现，提供Server端的系统服务
- # 三、[[AMS的启动过程]]
- # 四、AMS是什么？
  collapsed:: true
	- 1. 从java角度来看，ams就是一个java对象，实现了Ibinder接口，所以它是一个用于进程之间通信的接口，这个对象初始化是在systemServer.java 的run()
		- ```java
		  public Lifecycle(Context context) {
		      super(context);
		      mService = new ActivityManagerService(context);
		  }
		  ```
	- 2、AMS是一个服务
		- ActivityManagerService从名字就可以看出，它是一个服务，用来管理Activity，而且是一个系统服务，就是包管理服务，电池管理服务，震动管理服务等。
	- 3、AMS是一个Binder
	  collapsed:: true
		- ams实现了Ibinder接口，所以它是一个Binder，这意味着他不但可以用于进程间通信，还是一个线程，因为一个Binder就是一个线程。
		- 如果我们启动一个hello World安卓用于程序，里面不另外启动其他线程，这个里面最少要启动4个线程
		- 1 main线程
			- 只是程序的主线程，也是日常用到的最多的线程，也叫UI线程，因为android的组件是非线程安全的，所以只允许UI/MAIN线程来操作。
		- 2 GC线程
			- java有垃圾回收机制，每个java程序都有一个专门负责垃圾回收的线程，
		- 3 Binder1 就是我们的ApplicationThread
			- 这个类实现了Ibinder接口，用于进程之间通信，具体来说，就是我们程序和AMS通信的工具
		- 4 Binder2 就是我们的ViewRoot.W对象
			- 他也是实现了IBinder接口，就是用于我们的应用程序和wms通信的工具。
			- wms就是WindowManagerServicer ，和ams差不多的概念，不过他是管理窗口的系统服务
			- ```java
			  public class ActivityManagerService extends IActivityManager.Stub
			  implements Watchdog.Monitor, BatteryStatsImpl.BatteryCallback {}
			  ```
- # 五、[[AMS相关重要类]]
- # 六、[[Activity启动流程]]
- # 七、[[]]
-
- # 面试题
	- [[AMS面试题]]
	- [Android面试题——高级开发面试题一](https://blog.csdn.net/Calvin_zhou/article/details/128123302)