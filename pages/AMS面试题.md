# 2、[[AMS的启动流程-面试]]
- # 1、[[Activity启动流程]]api 28#card
  id:: 64fa8390-0f1e-41aa-92ca-3debd80dd319
  collapsed:: true
	- > Launcher点击图标 冷启动为例，主要分为三个阶段
	- ## ==**1、Launcher通知AMS，启动目标Activity**==
		- 1、Launcher也是一个App，调用==**startActivity**==方法启动目标APP的启动页，最终调用的是[[#red]]==**Instrumentation**==的[[#red]]==**execStartActivity**==方法
		- 2、该方法内部通过==**ServiceManager.getService**==。从SM中==**获取到AMS的代理对象**==（AIDL的Stub.asInterface方式），调用其startActivity的方法。（这里通过拿到AMS binder的代理对象。进行IPC通信的）[[#red]]==**这里AMS会验证目标Activity是否注册了**==
		- 3、这样启动Activity的工作就从Launcher交给了 AMS，AMS会判断目标APP进程是否存在。首次启动不存在，进入第二阶段
	- ## ==**2、AMS请求Zygote进程，fork app进程**==
		- 1、AMS 和 Zygote进程建立==**socket通信，告知Zygote创建fork子进程**==
		- 2、App进程fork完毕后，反射加载[[#red]]==**app入口函数：ActivityThread的main方法**==
		- 3、main()函数==**创建app主线程ActivityThread，并调用attach**==方法
		- 4、attach方法内调用[[#red]]==**AMS.attachApplication将ActivityThread的内部类ApplicationThread对象绑定至AMS**==，这样AMS就可以通过这个应用进程的代理对象来控制应用进程
	- ## ==**3、AMS通知App绑定Application（bindApplication）并启动Activity**==
		- 1、上述AMS拿到了app进程的代理对象。就可以进行跨进程调用==**ApplicationThread.bindApplication(）通知app进程去创建Instrumentacion和Application**==
		- 2、Application创建后会执行Application.attach()会触发其attachBaseContext(context)方法
		- 3、创建完Application后，==**AMS创建启动Activity的事务**==，发送给app进程
		- 4、ActivityThread解析完事务，最终调用到==**ActivityThread.performLaunchActivity**==.
		- 5、通过nstrumentation.newActivity()，执行attach方法
		- 6、mInstrumentation.callActivityOnCreate()：先调用activity.performCreate()，再回调Activity.onCreate()。
- # 3、App进程和AMS是如何通信的？#card
  id:: 64fa8390-c7a7-49aa-8558-21790ce31210
	- AMS所在的system Server进程  和  app进程在通过Binder互相通信时，实际上都是通过两者的代理类进行通信的。
	- 1、==**app进程通过ServiceManager.getService。**==从SM中获取到AMS的代理对象（AIDL的Stub.asInterface方式）
	- 2、app进程fork完毕后，会调用[[#red]]==**AMS.attachApplication将ActivityThread的内部类ApplicationThread**==对象绑定至AMS，这样AMS就可以通过这个代理对象来控制应用进程
- # 4、AMS和Launcher是怎么通信的？#card
  id:: 64fa8390-b7d7-4b88-a931-b2dd48e20e1a
	- 1、Launcher调用StartActivity，也是调用的是[[#red]]==**Instrumentation**==的[[#red]]==**execStartActivity**==方法
	- 2、该方法内部通过ServiceManager.getService。从SM中获取到AMS的代理对象（AIDL的Stub.asInterface方式），调用其startActivity的方法。（这里通过拿到AMS binder的代理对象。进行IPC通信的）
- # 5、Zygote和AMS是如何通信的？#card
  id:: 64fa8390-2d32-40e9-bf52-7cf02cb53c17
	- AMS和Zygote建立Socket连接，然后发送创建应用进程的请求
- # 3、AMS是什么，简单介绍下？#card
  id:: 64ab8ebf-6a04-4dd3-b0f1-8c4785915d2d
	- 1、Framework层，Android核心服务,全称为ActivityManagerService
	- 2、负责==**调度各应用进程,管理四大组件**==.实现了IActivityManager接口,应用进程能通过Binder机制调用系统服务
- # 4、Instrumentation是什么#card
  id:: 64f481f9-f206-4322-b352-884ce7371b3f
	- 1、Instrumentation会在应用程序的任何代码运行之前被实例化,
	- 2、它能够允许你监视[[#red]]==**应用程序和系统的所有交互**==，**==相当于提供一套hook方法==**
	- 3、它还会[[#red]]==**构造Application,构建Activity,以及生命周期都会经过这个对象去执行**==；
- # 5、Instrumentation和ActivityThread是什么关系
	- AMS与ActivityThread之间诸如Activity的创建、暂停等的交互工作都是由Instrumentation操作的。
	- 每个Activity都持有一个Instrumentation对象的一个引用， [[#red]]==**整个app进程中是只有一个Instrumentatio**==n。
	- 当startActivityForResult()调用之后，实际上还是调用了mInstrumentation.execStartActivity()。
- # 3、Activity的启动模式和flags在哪里设置的？#card
  id:: 64f68745-cafa-45da-8dfb-dc943b83799c
	- Activity的启动流程中（第一阶段）[[#red]]==**ActivityStarter的 startActivityUnchecked**==方法中
	  collapsed:: true
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
- # 4、应用程序的入口是哪里#card
  id:: 64ab8ec0-7522-46c3-8321-2b677edd2a02
	- ActivityThread.main() 应用启动入口
- # 5、ActivityThread和ApplicationThread关系和区别#card
  id:: 64f48525-92f7-42ec-a508-7774d1fc9055
	- ## 1、ActivityThread
		- ActivityThread在Android中代表Android的==**主线程**==，但是[[#red]]==**并不是一个Thread类**==。
		- ActivityThread类是Android 进程的初始类，它的==**main函数是这个App进程的入口**==。
		- 当创建完新进程之后，main函数被加载，然后执行一个loop的循环使当前线程进入消息循环
	- ## 2、ApplicationThread
		- ApplicationThread是ActivityThread的内部类，[[#red]]==**是一个Binder对象**==
		- 在此处它是作为IApplicationThread对象的server端等待client端的请求然后进行处理，最大的client就是AMS
- # 5、ActivityThread是主线程嘛？#card
  id:: 64abb677-f8b8-40d2-a604-d5422900dfe8
	- ActivityThread在Android中它就代表了Android的主线程，但是并不是一个Thread类。
	  严格来说，UI主线程不是ActivityThread。
	- ActivityThread类是Android APP进程的初始类，它的main函数是这个APP进程的入口。APP进程中UI事件的执行代码段都是由ActivityThread提供的。
- # ActivityManagerService是如何向zygote进程通信请求创建应用程序进程的?
	- 应用启动时，Launcher进程请求AMS，AMS发送创建应用进程请求，Zygote进程接受请求并fork应用进程。
	- 而AMS发送创建应用进程请求调用的是 ZygoteState.connect() 方法，ZygoteState 是 ZygoteProcess 的内部类。
	- Zygote 服务端接收到参数之后调用 ZygoteConnection.processOneCommand() 处理参数，并 fork 进程。
	- 最后通过 findStaticMain() 找到 ActivityThread 类的 main() 方法并执行，子进程就这样启动了
- # 6、 [[ContentProvider的onCreate执行时机]]#card
  id:: 64abb678-3e1f-418d-952d-2db8fb61b4c7
	- ContentProvider的OnCreate方法是在Application的attachBaseContext方法和onCreate之间执行的。
	  collapsed:: true
- # 7、Application是什么时候创建的，谁管理他的生命周期？#card
  id:: 64fa8390-b6c2-490e-955f-1c71ca6c59a6
	- 1、AMS向Zygote进程socket通信，[[#red]]==**创建应用程序进程后**==
	- 2、反射调用**==ActivityThread.main方法==**，经过进程间通信，和一系列调用会到
	- 3、调用到[[#red]]==**ActivityThread.handleBindApplication()**==，先创建Instrumentation,然后他反射创建Application，管理其生命周期、
- # 8、AMS的通信机制和作用？#card
  id:: 64abc32e-ad2f-4130-9c77-6498f8dcc7f4
	- binder通信机制
	- 作用：
		- 安全性考虑，binder机制中天然的带有调用者身份信息，uid
	- ##  [[AMS权限验证流程]]
- # 9、应用进程是什么时机创建的，如何和application绑定？
- [Android面试题——高级开发面试题一](https://blog.csdn.net/Calvin_zhou/article/details/128123302)
-