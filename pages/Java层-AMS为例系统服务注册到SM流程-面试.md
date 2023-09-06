## 详细见
	- ## [[以AMS为例注册服务到SM的流程-Java层]]
- ## 流程
	- ## 1、系统服务（SystemServer）会创建AMS实例==(AMS相当于客户端)==
	  collapsed:: true
		- 1、SystemServer反射创建ActivityManagerService.Lifecycle.class的实例，进而在构造函数创建AMS实例
	- ## 2、调用AMS的setSystemProcess，将自身AMS添加到SM中（不需要看下边）
	  collapsed:: true
		- 1、内部调用到getIServiceManager().addService()
		- 2、通过ProcessState创建BpBinder对象
		- 3、javaObjectForIBinder：创建BinderProxy 并和 BpBinder互相绑定
		- 4、ServiceManagerNative.asInterface 创建ServiceManagerProxy，并传入BinderProxy
		- ## 那么
		- 5、getIServiceManager()相当于 new ==ServiceManagerProxy==(new BinderProxy);
			- BinderProxy和 BpBinder
	- ## 3、实际执行的是ServiceManagerProxy.addService()（需要讲下边详细的）
		- 1、将AMS放入Binder机制的输入数据Parcel data中
		- 2、执行BinderProxy.transact方法传入ADD_SERVICE_TRANSACTION 注册服务的命令，并传入data数据
		- 3、Binder是通过命令封装来实现数据封装的。Native层会经过命令转换，让Binder驱动和服务端SM通信  处理添加服务，大致命令流程是
	- ## [[#red]]==命令流程 AMS添加服务举例，左侧客户端AMS 右侧服务端SM==
		- 1、客户端想添加服务，给Binder驱动发送**==BC_TRANSACTION==**命令
		- 2、Binder驱动收到这个命令后
			- 给Client发送消息[[#red]]==BR_TRANSACTION_COMPLETE==，让它休眠
			- 给Service发送消息[[#red]]==BR_TRANSACTION==,唤醒SM,让SM添加服务处理addService
		- 3、Service SM添加完服务AMS后，给Binder驱动发命令==**BC_REPLY**==
		- 4、Binder驱动收到这个命令后
			- 给Client发命令 [[#red]]==BR_REPLY==，唤醒客户端继续执行,客户端拿到BR_REPLY
			- 给Service SM发命令[[#red]]==BR_TRANSACTION_COMPLETE==,让它休眠
		-
-