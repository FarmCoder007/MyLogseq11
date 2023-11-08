## binder机制框架图#card
id:: 64aa14c8-d0b1-41b7-a3c5-c95f7a536d70
collapsed:: true
	- ![image.png](../assets/image_1688269631168_0.png){:height 657, :width 718}
- # 一、概念题
  collapsed:: true
	- ## 1、简单聊一聊Binder机制？#card
	  id:: 65367b9d-390c-410d-bdbd-4ec1208e3ffb
		- 1、Binder机制是一个==**基于Binder驱动和内核**==的机制，用于Android进程间通信机制。
		- 2、[[#red]]==**基于Client / Server架构的**==，服务端创建Binder对象，客户端创建Binder代理对象，通过底层binder驱动来间接调用服务端方法
		- 3、它通过进程间的[[#red]]==**共享内存（mmap内存映射）**==和==**句柄传递**==等方式==**实现数据的传输和共享**==。
	- ## 2、Binder机制的核心组件
	  collapsed:: true
		- ### [[Binder驱动]]：
		  collapsed:: true
			- Binder 驱动是 Android 系统的内核驱动程序，负责管理和处理进程间通信的底层细节。包括内存映射、句柄传递等。
		- ### [[Binder通信线程池]]：
		  collapsed:: true
			- Binder 通信线程池是一个系统级的线程池，负责处[[#red]]==**理进程间通信的请求和响应**==。
		- ### BinderProxy：Binder 代理对象：
		  collapsed:: true
			- Binder 代理对象是在[[#red]]==**客户端进程**==中生成的，用于代表服务端进程的对象。它封装了跨进程通信的细节，包括数据的封装和解析、方法的调用等。
			- **==通过代理对象，客户端可以方便地调用服务端的方法==**。
		- ### [[AIDL]]（Android Interface Definition Language）：
		  collapsed:: true
			- AIDL 是用于[[#red]]==**定义跨进程通信接口的语言**==。它提供了一种类似于接口定义语言（IDL）的方式，用于描述进程间通信接口中的方法和数据类型。AIDL 文件会被编译为 Java 接口和相应的代理类，以便在不同的进程间进行通信。
	- ## 3、[[Android中的Binder有什么优势？字节面试]]
	- ## 4、[[Binder怎么做到一次拷贝的？腾讯面试]]
	- ## 5、[[Mmap原理讲解=腾讯面试]]
	- ## 6、[[Binder传输数据的大小限制]]
	  collapsed:: true
		- Zygote孵化而来的用户进程，所映射的Binder内存大小 1M-8k
	- ## 7、 [[为什么Intent传输不能大于1M？]]
	- ## 8、[[系统服务与四大组件中服务的区别]]
	- ## 9、[[Binder线程、Binder主线程、Client请求线程的概念与区别]]
	- ## 10、[[Binder协议中BC与BR的区别]]
	- ## 11、Android APP进程天生支持Binder通信的原理是什么？
		- 1、Android APP进程都是由Zygote进程孵化出来的
		- 2、Zygote fork每个app子进程时，都会走onZygoteInit==**1、打开Binder设备**==。这时客户端就能通过binder进行远程通信，[[#red]]==**2、创建binder线程池**==。就具备了作为binder服务端的资格
		  id:: 64f691c7-7795-4822-96b9-c5961a9d6e55
		- 3、所以APP进程既能作为客户端，也能作为服务端
	- ## 12、Binder在传输数据的时候是如何层层封装的--不同层次使用的数据结构（==**命令的封装**==）
- # Binder机制的工作流程，CS架构，以应用进程和AMS交互为例
  collapsed:: true
	- 1、应用程序进程为客户端                           AMS为服务端
	-
	- ==**服务端进程**==创建一个==**Binder 对象**==
	- [[#red]]==**客户端进程**==通过绑定服务获取服务端的 Binder 对象，并[[#red]]==**生成BindProxy代理对象**==。
	- [[#green]]==**客户端通过代理对象调用服务端的方法**==，实际上是调用了代理对象中的方法。
	-
	- ==**代理对象**==将方法调用==**封装成一个请求**==，并通过 Binder 机制发送给服务端。
	- 服务端解析处理，将结果通过 Binder 机制返回给客户端进程。
	- 客户端进程接收到结果后，==**代理对象将结果解析**==并返回给调用方。
- # 二、Binder机制的整体框架
  collapsed:: true
	- ## 图解
	  collapsed:: true
		- ![image.png](../assets/image_1688269631168_0.png){:height 657, :width 718}
	- # Java层
		- 客户端：==**BinderProxy**== 为binder代理对象
		- 服务端：[[#red]]==**Binder**== 为binder对象
	- # Native层
		- 客户端：**==BpBinder==** 也为binder的代理对象
		- 服务端：[[#red]]==**BBinder**== 也为binder的对象
	- # 内核驱动层：binder对象用结构体表示的
		- 客户端：==**binder_ref**==:      代表binder引用
		- 服务端：[[#red]]==**binder_node**==:  代表Binder对象
- # 五、Binder机制框架每层的启动，源码相关
  collapsed:: true
	- ##  1、最底层内核层：[[内核层-Binder驱动启动流程]]
	- ## 2、Native层 SM：[[Binder机制大管家-SM的启动和获取流程-面试]]
	- ##### 3、JNI层（Native到java）[[JNI层-Binder的Jni方法注册流程-面试]]
	- ## 4、Java层AMS：[[Java层-AMS为例系统服务注册到SM流程-面试]]
- # ServiceManager相关
	- ## 1、ServiceManager介绍 / Binder机制中如何找到目标Binder的#card
	  id:: 64f68746-993d-4f36-99e3-0381464d3481
		- > ServiceManager==它充当了系统服务的注册和查找中心==。它负责管理和提供系统级服务给应用程序和其他系统组件使用
		- > [[#red]]==**Framework 层的ServiceManager 和 c++ 层 service_manager 实际上最终都对应handle = 0的 binder**==
		- 1、我们无法直接获取 AMS，PMS等Ibinder对象进行进程间通信，但是==**系统启动时，这些系统服务会自动注册到SM大管家中。**==
		- 2、我们可以[[#red]]==**直接获取handle = 0来获取SM，进而获取AMS等系统服务**==
	- ## 2、SM主要是系统服务的注册和查询，并不管app bind的service那么我们Android，四大组件中的bindService 绑定服务是怎么注册查询的？--app的是注册到AMS中管理的
		- 参考AIDL 或者 https://juejin.cn/post/6844903469988446221
	- ## 2、ServiceManager作用: 注册系统服务和查找系统服务
	- ## 3、[[SystemServiceManager和ServiceManager区别]]#card
	  id:: 64f68746-37c0-4795-abe0-dac8884b7a95
		- SystemServiceManager负责创建管理系统服务，比如AMS的创建，
		- ServiceManager是binder机制大管家，负责管理和提供系统服务对应的binder对象。
- # 参考
	- [听说你Binder机制学的不错，来面试下这几个问题](https://blog.csdn.net/idaretobe/article/details/128293503)
	- [听说你Binder机制学的不错，来解决下这几个问题（一）](https://juejin.cn/post/6844903469971685390)
	- [听说你 Binder 机制学的不错，来解决下这几个问题（二）](https://juejin.cn/post/6844903469984268296#heading-4)
	- [听说你 Binder 机制学的不错，来解决下这几个问题（三）](https://juejin.cn/post/6844903469988446221)
	- https://juejin.cn/post/6844903469971685390