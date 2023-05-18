- # 背景
  collapsed:: true
	- 近期应工信部要求整改，其中一项是的整改要求是隐私弹窗之前不允许收集个人信息，个人信息包括设备和个人的敏感信息，这个条款按照需求理解就是隐私弹框之前不允许有任何业务逻辑有采集信息行为，代码层理解就是隐私弹窗弹出前不允许调用任何其他逻辑，包括我们程序的初始化操作。因为我们程序的初始化之前是放在隐私弹窗之前开始执行的，那为迎合本次整改，需要将我们的初始化延迟到隐私政策同意之后执行，如何能快速实现，这个改造呢？借鉴了同城hook Instrumentation 的方案（朝彬、曾老师开发），实现方案不是本次要讲的内容，大家可以自行下来查看代码实现（PrivacyInstrumentation类），本次要讲的是app应用的启动流程，当你理解了本次所讲的内容，你再看实现方案就能更好的理解，从源码逐步分析启动流程的每个环节，涉及到每个类的作用，带大家一一了解。
- # 问题
  collapsed:: true
	- 本文会带大家了解以下问题：
	- 应用的进程是什么时机创建的，进程如何和application绑定？
	  app 应用启动过程中关键核心类的作用以及他们之间的关系？
	  Application是什么时候创建的，谁管理他的生命周期？
	  Activity的创建和生命周期是由谁管理？
	  AMS的通信机制和作用？
	  ContentProvider的onCreate()是有系统调用的，那什么时机执行呢？
	  若以上内容都已理解，那接下来的内容就当温习和探讨吧 ：）
- # 应用启动流程所涉及的核心类及作用
  collapsed:: true
	- 先简述一下启动过程中关键类及作用，便于大家理解下面的内容：
	- ActivityManagerService：Framework层，Android核心服务,简称AMS,负责调度各应用进程,管理四大组件.实现了IActivityManager接口,应用进程能通过Binder机制调用系统服务，以下简称AMS；
	  Instrumentation： Instrumentation会在应用程序的任何代码运行之前被实例化,它能够允许你监视应用程序和系统的所有交互.它还会构造Application,构建Activity,以及生命周期都会经过这个对象去执行；
	  ActivityThread：应用的主线程.程序的入口.在main方法中开启loop循环,不断地接收消息,处理任务.
	  ApplicationThread： 是ActivityThread的内部类，也是一个Binder对象,执行组件和Application 的调度任务。
	  ServiceManager：用于管理所有的系统服务，维护着系统服务和客户端的binder通信。
	  TaskRecord：Activity的管理者，一个ActivityRecord对应一个Activity，保存了一个Activity的所有信息;但是一个Activity可能会有多个ActivityRecord,因为Activity可以被多次启动，这个主要取决于其启动模式。
	  ActivityStack：Stack的管理者，用来管理TaskRecord的，包含了多个TaskRecord
	  ActivityStackSupervisor：ActivityStack的管理者，早期的Android版本是没有这个类的，直到Android 6.0才出现。
- # 启动流程
	- ## 本次分析的环境条件：
		- 1. 源码基于android 28
		  2. 本次启动分析是基于Launcher的桌面启动应用
		  3. AOSP源码阅读可使用的方式：
		      3.1 在线阅读 https://cs.android.com 、https://github.com/aosp-mirror（google已经把framework源码托管在了gitHub） 或http://androidxref.com/（快速搜索引擎） 在线搜索及浏览 。
		      3.2 离线阅读工具OpenGrok、Source Insight、AS或我们常用Sublime Text；
		      3.3 源码下载地址：
		          3.3.1 [国内GitHub地址](https://github.com/aosp-mirror/platform_frameworks_base)
		          3.3.2 [清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)
		          3.3.3 [中国科学技术大学开源软件镜像站](http://mirrors.ustc.edu.cn/)
		          3.3.4 若只阅读Android platform 源码，可以从科大上clone：
		  git clone git://mirrors.ustc.edu.cn/aosp/platform/frameworks/base --depth=1
		  因为AOSP资源较大，本次采用在线方式阅读分析
	- ## 启动入口Launcher.StartActivity
		- Launcher,我们安卓桌面,他也是一个应用.只不过他有些特殊属性,特殊在于它是系统开机后第一个启动的应用,并且该应用常驻在系统中,不会被杀死,用户一按home键就会回到桌面(回到该应用).桌面上面放了很多很多我们自己安装的或者是系统自带的APP,我们通过点击这个应用的快捷方法可以打开我们的应用，桌面Launcher也是一个Activity，也是通过startActivity打开我们的应用的，可以认为我们的应用都是通过其他应用打开的。
	- ## Activity的startActivity方法
	  collapsed:: true
		- 代码
		  collapsed:: true
			- ```
			  @Override
			  public void startActivity(Intent intent, @Nullable Bundle options) {
			      if (mIntent != null && mIntent.hasExtra(AutofillManager.EXTRA_RESTORE_SESSION_TOKEN)
			              && mIntent.hasExtra(AutofillManager.EXTRA_RESTORE_CROSS_ACTIVITY)) {
			          if (TextUtils.equals(getPackageName(),
			                  intent.resolveActivity(getPackageManager()).getPackageName())) {
			              ...
			          }
			      }
			      if (options != null) {
			          startActivityForResult(intent, -1, options);
			      } else {
			          // Note we want to go through this call for compatibility with
			          // applications that may have overridden the method.
			          startActivityForResult(intent, -1);
			      }
			  }
			  ```
		- startActivity会调用startActivityForResult方法。
		  collapsed:: true
			- ```
			  public void startActivityForResult(@RequiresPermission Intent intent, int requestCode,
			          @Nullable Bundle options) {
			      if (mParent == null) {
			          options = transferSpringboardActivityOptions(options);
			          //调用Instrumentation.execStartActivity()启动新的Activity。mMainThread类型为ActivityThread, 在attach()函数被回调时被赋值
			          Instrumentation.ActivityResult ar =
			              mInstrumentation.execStartActivity(
			                  this, mMainThread.getApplicationThread(), mToken, this,
			                  intent, requestCode, options);
			          if (ar != null) {
			              // 如果activity之前已经启动，而且处于阻塞状态，execStartActivity函数直接返回要启动的activity的result或者null。
			              //如果结果不为空，发送结果
			              mMainThread.sendActivityResult(
			                  mToken, mEmbeddedID, requestCode, ar.getResultCode(),
			                  ar.getResultData());
			          }
			          if (requestCode >= 0) {
			              // 如果需要一个返回结果，那么赋值mStartedActivity为true，在结果返回来之前保持当前Activity不可见
			              mStartedActivity = true;
			          }
			  
			          cancelInputsAndStartExitTransition(options);
			          // TODO Consider clearing/flushing other event sources and events for child windows.
			      } else {
			          if (options != null) {
			              mParent.startActivityFromChild(this, intent, requestCode, options);
			          } else {
			              // Note we want to go through this method for compatibility with
			              // existing applications that may have overridden it.
			              mParent.startActivityFromChild(this, intent, requestCode);
			          }
			      }
			  }
			  ```
		- mParent是ActivityGroup，之前的组合模式，已经不推荐使用了，现在被Fragment替代，一般情况为空，所以都会进入为空判断分支，随后调用了Instrumentation的execStartActivity方法,其中mMainThread是ActivityThread(就是从这里开始启动一个应用的),mMainThread.getApplicationThread()是获取ApplicationThread，ApplicationThread是ActivityThread的内部类,下文介绍。
	- ## Instrumentation执行execStartActivity方法
	  collapsed:: true
		- 代码
		  collapsed:: true
			- ```
			  @UnsupportedAppUsage	
			  public ActivityResult execStartActivity(
			          Context who, IBinder contextThread, IBinder token, Activity target,
			          Intent intent, int requestCode, Bundle options) {
			      //1. 将ApplicationThread转为IApplicationThread
			      IApplicationThread whoThread = (IApplicationThread) contextThread;
			      Uri referrer = target != null ? target.onProvideReferrer() : null;
			      if (referrer != null) {
			          intent.putExtra(Intent.EXTRA_REFERRER, referrer);
			      }
			      ...
			      try {
			  
			          intent.migrateExtraStreamToClipData(who);
			          intent.prepareToLeaveProcess(who);
			          //2. 获取AMS实例,调用startActivity方法
			          int result = ActivityManager.getService().startActivity(whoThread,
			                  who.getBasePackageName(), who.getAttributionTag(), intent,
			                  intent.resolveTypeIfNeeded(who.getContentResolver()), token,
			                  target != null ? target.mEmbeddedID : null, requestCode, 0, null, options);
			          checkStartActivityResult(result, intent);
			      } catch (RemoteException e) {
			          throw new RuntimeException("Failure from system", e);
			      }
			      return null;
			  }
			  ```
		- IApplicationThread是一个Binder接口,继承自android.os.IInterface接口，IInterface接口是Binder的基础类，ApplicationThread是继承了IApplicationThread.Stub(由Android SDK工具会生成以IApplicationThread.aidl 文件命名的 .java 接口文件，生成的接口包含一个名为 Stub 的子类),实现了IApplicationThread的,所以可以转成IApplicationThread.由ApplicationThread和AMS通信进一步执行启动流程。
	- ## ServiceManager获取ActivityManagerService（AMS）并继续启动startActivity
	  collapsed:: true
		- 在Instrumentacion的execStartActivity方法中ActivityManager.getService()获取ActivityManagerService（AMS）实例,调用AMS的startActivity方法，看一下ActivityManagerService的获取方式，通过ServiceManager.getService(Context.ACTIVITY_SERVICE)
		  collapsed:: true
			- ```
			  @UnsupportedAppUsage
			  public static IActivityManager getService() {
			      return IActivityManagerSingleton.get();
			  }
			  @UnsupportedAppUsage
			  private static final Singleton<IActivityManager> IActivityManagerSingleton =
			          new Singleton<IActivityManager>() {
			              @Override
			              protected IActivityManager create() {
			                  //System service 获取AMS 
			                  final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE);
			                  final IActivityManager am = IActivityManager.Stub.asInterface(b);
			                  return am;
			              }
			          };
			  ```
		- ServiceManager是安卓中一个重要的类，用于管理所有的系统服务，维护着系统服务和客户端的binder通信。返回的是Binder对象,用来进行应用与系统服务之间的通信的.
		- ServiceManager存在于System Server进程，是由Zygote进程fork而来，Zygote孵化的第一个进程，System Server负责启动和管理整个Java framework，包含ActivityManager，WindowManager，PackageManager，PowerManager等服务，AMS就是其中的一个重要服务
		  collapsed:: true
			- ![image.png](../assets/image_1684416825352_0.png)
		- 对于用户空间，不同进程之间彼此是不能共享的，而内核空间却是可共享的。Client进程向Server进程通信，恰恰是利用进程间可共享的内核内存空间来完成底层通信工作的，Client端与Server端进程往往采用ioctl等方法跟内核空间的驱动进行交互。ioctl是设备驱动程序中对设备的I/O通道进行管理的函数，Linux设备控制接口采用的就是这个函数。用户空间和内核空间合起来就是操作系统的虚拟地址空间（线性地址空间），针对32位 Linux 操作系统而言，最高的 1G 字节(从虚拟地址 0xC0000000 到 0xFFFFFFFF)由内核使用，称为内核空间。而较低的 3G 字节(从虚拟地址 0x00000000 到 0xBFFFFFFF)由各个进程使用，称为用户空间。
		- 操作系统的核心是内核，它独立于普通的应用程序，可以访问受保护的内存空间，也有访问底层硬件设备的所有权限。为了保证内核的安全，现在的操作系统一般都强制用户进程不能直接操作内核。系统资源管理都是在内核空间中完成的。比如读写磁盘文件，分配回收内存，从网络接口读写数据等等。我们的应用程序是无法直接进行这样的操作的。但是我们可以通过内核提供的接口来完成这样的任务。比如应用程序要读取磁盘上的一个文件，它可以向内核发起一个 "系统调用" 告诉内核："我要读取磁盘上的某某文件"。
		- 接着讲解ServiceManager
			- ```
			  ```