# 1、[[Activity启动流程]]
- # 2、[[AMS的启动流程-面试]]
- # 3、AMS是什么，简单介绍下？
	- 1、Framework层，Android核心服务,全称为ActivityManagerService
	- 2、负责调度各应用进程,管理四大组件.实现了IActivityManager接口,应用进程能通过Binder机制调用系统服务
- # 4、Instrumentation是什么
  id:: 64f481f9-f206-4322-b352-884ce7371b3f
	- 1、Instrumentation会在应用程序的任何代码运行之前被实例化,
	- 2、它能够允许你监视[[#red]]==**应用程序和系统的所有交互**==，相当于提供一套hook方法
	- 3、它还会构造Application,构建Activity,以及生命周期都会经过这个对象去执行；
- # 5、Instrumentation和ActivityThread是什么关系
	- AMS与ActivityThread之间诸如Activity的创建、暂停等的交互工作都是由Instrumentation操作的。
	- 每个Activity都持有一个Instrumentation对象的一个引用， 整个进程中是只有一个Instrumentation。
	- 当startActivityForResult()调用之后，实际上还是调用了mInstrumentation.execStartActivity()。
- # 3、Activity的启动模式和flags在哪里设置的？
  collapsed:: true
	- Activity的启动流程中（第一阶段）ActivityStarter的 startActivityUnchecked方法中
		- ```java
		  
		  // 1、根据启动的intent 识别启动模式，并处理
		  // 2、判断启动模式并在flags上追加对应的标记
		  private int startActivityUnchecked(final ActivityRecord r, ActivityRecord sourceRecord,
		          IVoiceInteractionSession voiceSession, IVoiceInteractor voiceInteractor,
		          int startFlags, boolean doResume, ActivityOptions options, TaskRecord inTask,
		          ActivityRecord[] outActivity) {
		      .......
		      mSupervisor.resumeFocusedStackTopActivityLocked(mTargetStack, mStartActivity, mOptions);
		  
		      return START_SUCCESS;
		  }
		  ```
	- 的setInitialState中处理 启动模式相关
- # 4、应用程序的入口是哪里
	- ActivityThread.main() 应用启动入口
- # 5、ActivityThread和ApplicationThread关系和区别
  id:: 64f48525-92f7-42ec-a508-7774d1fc9055
	- ## 1、ActivityThread
		- ActivityThread在Android中代表Android的==**主线程**==，但是[[#red]]==**并不是一个Thread类**==。
		- ActivityThread类是Android 进程的初始类，它的==**main函数是这个App进程的入口**==。
		- 当创建完新进程之后，main函数被加载，然后执行一个loop的循环使当前线程进入消息循环
	- ## 2、ApplicationThread
		- ApplicationThread是ActivityThread的内部类， 是一个Binder对象
		- 在此处它是作为IApplicationThread对象的server端等待client端的请求然后进行处理，最大的client就是AMS
- # 5、ActivityThread是主线程嘛？
	- ActivityThread在Android中它就代表了Android的主线程，但是并不是一个Thread类。
	  严格来说，UI主线程不是ActivityThread。
	- ActivityThread类是Android APP进程的初始类，它的main函数是这个APP进程的入口。APP进程中UI事件的执行代码段都是由ActivityThread提供的。
- # ActivityManagerService是如何向zygote进程通信请求创建应用程序进程的?
	- 应用启动时，Launcher进程请求AMS，AMS发送创建应用进程请求，Zygote进程接受请求并fork应用进程。
	- 而AMS发送创建应用进程请求调用的是 ZygoteState.connect() 方法，ZygoteState 是 ZygoteProcess 的内部类。
	- Zygote 服务端接收到参数之后调用 ZygoteConnection.processOneCommand() 处理参数，并 fork 进程。
	- 最后通过 findStaticMain() 找到 ActivityThread 类的 main() 方法并执行，子进程就这样启动了
- # 6、 [[ContentProvider的onCreate执行时机]]
- # 7、Application是什么时候创建的，谁管理他的生命周期？
	- 1、AMS向Zygote进程socket通信，创建应用程序进程后
	- 2、反射调用ActivityThread.main方法，经过进程间通信，和一系列调用会到
	- 3、调用到ActivityThread.handleBindApplication()，先创建Instrumentation,然后他反射创建Application，管理其生命周期、
- # 8、AMS的通信机制和作用？
	- binder通信机制
	- 作用：
		- 安全性考虑，binder机制中天然的带有调用者身份信息，uid
	- ##  [[AMS权限验证流程]]
- # 9、应用进程是什么时机创建的，如何和application绑定？
- [Android面试题——高级开发面试题一](https://blog.csdn.net/Calvin_zhou/article/details/128123302)
-