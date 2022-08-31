- # 一、简介
	- SystemServer 进程主要用于创建系统服务，我们熟知的 AMS WMS PMS 都是由它来创建的，#
- # 二、Zygote 如何处理SystemServer 进程的
	- ![image.png](../assets/image_1660120698552_0.png){:height 601, :width 746}
	- 1、在Zygotelnit. java 的startSystemServer 方法中启动了 SystemServer ，
	- 2、handleSystemServerProcess 方法来启动 SystemServer 进程。
- # 三、部分系统服务及其作用
	- ![image.png](../assets/image_1660121188436_0.png)
- # 四、职责
	- SystemServer 进程被创建后，主要做了如下工作：
	  (1）启动 Binder 线程池，这样就可以与其他进程进行通信
	  (2）创建 SystemServiceManager ，其用于对系统的服务[具体见上表]进行创建、启动和生命周期管理。
	  (3）启动各种系统服务。
-