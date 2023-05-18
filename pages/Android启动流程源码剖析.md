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
			- ```
			  ```