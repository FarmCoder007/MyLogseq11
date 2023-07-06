- ## Android系统服务
	- 系统服务一般是在系统启动的时候，由SystemServer进程创建并注册到ServiceManager中的
- ## 普通的服务Service
	- 普通服务一般是通过ActivityManagerService启动的服务，或者说通过四大组件中的Service组件启动的服务
- # 二者区别
  collapsed:: true
	- ## 1、启动上
	  collapsed:: true
		- ## 系统服务
			- 一般都是SystemServer进程负责启动，比如AMS，WMS，PKMS，电源管理等，这些服务本身其实实现了Binder接口，作为Binder实体注册到ServiceManager中，被ServiceManager管理，而SystemServer进程里面会启动一些Binder线程，主要用于监听Client的请求，并分发给响应的服务实体类，可以看出，这些系统服务是位于SystemServer进程中（有例外，比如Media服务）。
		- ## bindService类型的服务
			- 这类服务一般是通过Activity的startService或者其他context的startService启动的，这里的Service组件只是个封装，主要的是里面Binder服务实体类，这个启动过程不是ServcieManager管理的，而是通过ActivityManagerService进行管理的，同Activity管理类似
	- ## 2、服务的注册与管理
	  collapsed:: true
		- ![](https://img-blog.csdnimg.cn/e8bb2eb6c807432ab7610b56a4637bc4.png)
		- 系统服务
			- 一般都是通过ServiceManager的addService进行注册的，这些服务一般都是需要拥有特定的权限才能注册到ServiceManager
		- bindService启动的服务
			- 可以算是注册到ActivityManagerService，只不过ActivityManagerService管理服务的方式同ServiceManager不一样，而是采用了Activity的管理模型
	- ## 3、使用方式
		- 使用系统服务
			- 一般都是通过ServiceManager的getService得到服务的句柄，这个过程其实就是去ServiceManager中查询注册系统服务
		- bindService启动的服务，
			- 主要是去ActivityManagerService中去查找相应的Service组件，最终会将Service内部Binder的句柄传给Client。