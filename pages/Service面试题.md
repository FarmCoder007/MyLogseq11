# 1、Service是什么？#card
id:: 64f6d703-9751-4973-bf13-72885fa2bafa
	- 1、Service是Android实现 程序后台运行的解决方案
	  id:: 2e594fe7-f80d-46fd-ae6d-09fd6c625c01
	- 2、它非常适合去[[#red]]==**执行那些不需要和用户交互而且还要长期运行的任务**==。
	- 3、代码运行在 主线程， 不能进行耗时操作，需要在清单文件注册
	  id:: 64f6d72a-40a0-4e9a-934a-2c6cc474c20d
- # 1、Broadcast receiver {{cloze 不能}}启动和绑定Service
  id:: 64aa14c9-803d-4612-b3a0-f34e349d0569
- # 2、startService()与bindService()可以同时执行吗，怎么结束？#Card
  id:: 64aa14c9-076b-467c-9257-b908eafe5100
	- 并不冲突，同一个service可能既有组件调用了startService()启动它，又有组件与它进行了绑定。
	- 当同一个service与其他组件同时存在这两种联系时，其生命周期会发生变化，必须从两种方法的角度看service均停止才能真正停止。
	- stopService()+UnbinderService
- # 3、Service的生命周期#Card
  id:: 64f6d844-8c40-4ea8-aea0-dace6a330927
  collapsed:: true
	- ![image.png](../assets/image_1693898347377_0.png)
	- ## onCreate()
		- 服务创建的时候调用
	- ## [[onStartCommand()的三种返回值]]
	  id:: 26993eb6-6d3f-4c75-a97f-4cd8b050aed7
	- ## onBind()
		- 当其他组件通过bindService()方法与service相绑定之后，此方法将会被调用。这个方法有一个IBinder的返回值，这意味着在重写它的时候必须返回一个IBinder对象，它是用来支撑其他组件与service之间的通信的——另外，如果你不想让这个service被其他组件所绑定，可以通过在这个方法返回一个null值来实现。
	- ## onUnbind()
	- ## onDestory()
		- 服务销毁的时候调用
- # 3、[[startService和bindService区别]]
- # 4、[[onStartCommand()的三种返回值]]
- # 5、[[绑定服务中 ServiceConnection的参数解析]]