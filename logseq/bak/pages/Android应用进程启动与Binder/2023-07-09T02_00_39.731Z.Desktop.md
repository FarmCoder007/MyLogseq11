- # 一、应用进程启动
  collapsed:: true
	- ## linux进程启动的方式
	  collapsed:: true
		- 启动进程有两种方式。调用fork函数启动子进程，pid为0时代表在子进程返回值，否则是父进程。左边启动方式，默认继承父进程的系统资源。右边execve方式传入启动参数，父进程资源全被清空掉。Android系统采用了Linux内核，因此，Android应用进程的创建与Linux如出一辙。
		  ps: path代表二进制程序的路径，argv代表需要设置的参数，env代表需要设置的环境变量。
		- ![image.png](../assets/image_1684413869562_0.png){:height 306, :width 746}
	- ## Android应用进程启动的基本流程
		- 在这里要明白两个问题:
		- 1、什么时候才会触发进程启动？
		  2、进程是谁来启动？怎么启动？
		- Android系统里面没有接口可以调用去主动启动一个进程，全是被动的启动。启用组件的时候，发现组件发在的进程不存在，才会去启动进程。这些功能都是在Framework中去做的，AMS在此流程中扮演了很重要的的角色。
		- ![image.png](../assets/image_1684413896203_0.png){:height 161, :width 746}
		- app.thread != null，这是判断IApplicationThread（AMS拿到的应用binder句柄),保证实现双向的binder调用。
		- ![image.png](../assets/image_1684413907711_0.png)
		- 那IApplicationThread句柄是怎么传给AMS的呢？AMS通过配置Zygote启动的参数，最终通过socket写入到Zygote进程，Zygote进程收到socket请求后会处理请求参数，同时对于该进程，也指定了一个类作为执行的入口类，这个类就是ActivityThread。ActivityThread中main方法，作为应用进程的入口，所有的应用进程的初始化都会走这重要的四行代码。
		- ![image.png](../assets/image_1684413923637_0.png)
		- 因此，对一个应用进程启动完成的标志就是以下两点：
		  1、AMS向Zygote发起启动进程的请求,zygote启动完进程之后会给AMS返回进程pid。
		  2、应用启动后，向AMS注册IApplicationThread。
		- 会不会有一种可能，进程已经启动，但是还没有来得及注册Thread,然后另外一个组件去启动，会不会重复启动进程呢？
		- ![image.png](../assets/image_1684413940112_0.png)
		-
	-
	-
	-
	-
	-
	-
- # 二、应用中启用的binder机制
	- ## 在何处启用binder机制
	  collapsed:: true
		- startProcessLocked方法除了判断进程判断，还会负责进程启动的核心逻辑。
		  collapsed:: true
			- ![image.png](../assets/image_1684413978133_0.png){:height 344, :width 453}
		- AMS通过Socket与Zygote发送请求参数，Zygote收到socket请求后会通过ZygoteConnection，核心逻辑在runOnce方法中，会调用handleChildProc。
			- ![image.png](../assets/image_1684413994557_0.png){:height 250, :width 716}
		- handleChildProc方法调用了ZygoteInit的zygoteInit方法，核心逻辑有以下三步：
		  1、启动binder线程池；
		  2、读取请求参数拿到ActivityThread类并执行他的main函数，执行thread.attach告知AMS并回传自己的binder句柄；
		  3、执行Looper.loop启动消息循环；
	- ## 怎么启用binder机制
	  collapsed:: true
		- 依然是在handleChildProc中，从nativeZygoteInit方法中查看如何启动binder。启用binder机制的过程可以分为两部分。
			- 第一部分：打开binder驱动，映射内存，分配缓冲区。
			  collapsed:: true
				- ![image.png](../assets/image_1684414046327_0.png)
			- 第二部分：注册binder线程，进入binder loop。
			  collapsed:: true
				- ![image.png](../assets/image_1684414069027_0.png)
- # 一张图做个了结
	- ![image.png](../assets/image_1684414085853_0.png){:height 317, :width 746}
	- Android应用进程的启动可以总结成以下步骤：
	  1、AMS判断APP进程是否存在，不存在则发起socket请求。
	  2、Zygote进程接收请求并处理参数。
	  3、Zygote进程fork出应用进程，应用进程继承得到虚拟机实例。
	  4、应用进程启动binder线程池、运行ActivityThread类的main函数、启动Looper循环。