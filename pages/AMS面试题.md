- # 1、[[Activity启动流程]]
- # 2、[[AMS的启动流程-面试]]
- # 3、Activity的启动模式和flags在哪里设置的？
	- Activity的启动流程中（第一阶段）ActivityStarter的 startActivityUnchecked方法中
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
- # 4、应用程序的入口是哪里
	- ActivityThread.main() 应用启动入口
- # 5、ActivityThread是主线程嘛？
	- ActivityThread在Android中它就代表了Android的主线程，但是并不是一个Thread类。
	  严格来说，UI主线程不是ActivityThread。
	- ActivityThread类是Android APP进程的初始类，它的main函数是这个APP进程的入口。APP进程中UI事件的执行代码段都是由ActivityThread提供的。
- # 6、 [[ContentProvider的onCreate执行时机]]
- # 7、Application是什么时候创建的，谁管理他的生命周期？
  collapsed:: true
	- 1、AMS向Zygote进程socket通信，创建应用程序进程后
	- 2、反射调用ActivityThread.main方法，经过进程间通信，和一系列调用会到
	- 3、调用到ActivityThread.handleBindApplication()，先创建Instrumentation,然后他反射创建Application，管理其生命周期、
- # 8、AMS的通信机制和作用？
	- binder通信机制
	- 作用：
		- 安全性考虑，binder机制中天然的带有调用者身份信息，uid
	- ##  [[AMS权限验证流程]]
		-
- [Android面试题——高级开发面试题一](https://blog.csdn.net/Calvin_zhou/article/details/128123302)
-