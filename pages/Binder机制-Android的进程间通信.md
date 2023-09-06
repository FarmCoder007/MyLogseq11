# 一、什么是Binder机制
title:: Binder机制-Android的进程间通信
collapsed:: true
	- ## 1、是Android进程间通信机制
	  collapsed:: true
		- ## Binder框架
			- ![image.png](../assets/image_1688269631168_0.png){:height 657, :width 718}
			- # 客户端
				- ## Java层
					- BinderProxy 为binder代理对象
				- ## native 层
					- BpBinder 也为binder的代理对象
			- # 服务端
				- ## Java层
					- Binder 为binder对象
				- ## native 层
					- BBinder 也为binder的对象
			- # 驱动层
				- 也有一个代表Binder的代理对象，一个代表binder对象。用结构体表示的
				- binder_node:  代表Binder对象
				- binder_ref:      代表binder引用
		-
	- ## 2、内核层有一个[[Binder驱动]]设备
	- ## 3、Java里有个Binder.java 是实现了Ibinder接口-实现了跨进程能力
- # 二、Binder机制的核心组件
  collapsed:: true
	- ## Binder：
		- Binder 是一个基于驱动和内核的机制，用于进程间通信。它通过进程间的共享内存和句柄传递等方式实现数据的传输和共享。
	- ## [[Binder驱动]]：
		- Binder 驱动是 Android 系统的内核驱动程序，负责管理和处理进程间通信的底层细节。它提供了进程间通信所需的底层机制，包括内存映射、句柄传递等。
	- ## [[Binder通信线程池]]：
		- Binder 通信线程池是一个系统级的线程池，负责处理进程间通信的请求和响应。它通过多线程的方式处理并发的通信请求，提高了通信的效率。
	- ## BinderProxy：Binder 代理对象：
		- Binder 代理对象是在客户端进程中生成的，用于代表服务端进程的对象。它封装了跨进程通信的细节，包括数据的封装和解析、方法的调用等。通过代理对象，客户端可以方便地调用服务端的方法。
	- ## [[AIDL]]（Android Interface Definition Language）：
		- AIDL 是用于定义跨进程通信接口的语言。它提供了一种类似于接口定义语言（IDL）的方式，用于描述进程间通信接口中的方法和数据类型。AIDL 文件会被编译为 Java 接口和相应的代理类，以便在不同的进程间进行通信。
- # 三、Binder 机制的工作流程如下
  id:: 64a3d01e-70d9-4848-b90c-65f40939475f
	- 服务端进程创建一个 Binder 对象，并将其作为服务的实例返回给客户端进程。
	- 客户端进程通过绑定服务的方式获取到服务端的 Binder 对象，并生成相应的代理对象。
	- 客户端通过代理对象调用服务端的方法，实际上是调用了代理对象中的方法。
	- 代理对象将方法调用封装成一个请求，并通过 Binder 机制将请求发送给服务端进程。
	- 服务端进程接收到请求后，Binder 机制将请求解析并转发给实际的服务对象进行处理。
	- 服务对象处理完请求后，将结果通过 Binder 机制返回给客户端进程。
	- 客户端进程接收到结果后，代理对象将结果解析并返回给调用方。
- # 四、[[Binder类图方法关系]]
- # 五、Binder框架分层介绍
	- ![image.png](../assets/image_1688269631168_0.png){:height 657, :width 718}
	- # [[JAVA层服务的注册和获取]]Java层以AMS举例
	- # [[Binder的Jni方法注册流程，JNI层代码]]JNI层
	- # [[ServiceManager启动和获取-Native层]]native层
	- #  [[Binder驱动]]内核层
- # 六、[[ServiceManager]]
-
-
- # [[Binder机制面试题]]
  collapsed:: true
	- # [[Binder怎么做到一次拷贝的？腾讯面试]]
	- # [[为什么Intent传输不能大于1M？]]
	- # [[Mmap原理讲解=腾讯面试]]
	- # [[Android中的Binder有什么优势？字节面试]]
	- # [[Binder传输数据的大小限制]]