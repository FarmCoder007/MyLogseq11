- ## 详细见 [[Binder驱动是怎么启动的？内核层的代码]]
-
- # 前言
	- 1、在linux中一切皆文件，binder驱动也是文件，都是可读写的
	- 2、而mmap就是将虚拟内存和文件联系起来的，打开binder驱动，涉及mmap为binder驱动开辟虚拟内存空间
	- 3、Binder是启动的misc（注册简单）设备(binder是没有硬件的，binder就相当于一块内存)
- # 大概流程
	- ### 1、通过 (内核层的binder.c)  binder_init（），创建初始化/dev/binder设备节点
	  collapsed:: true
		- 1、创建名为binder的单线程的工作队列
		- 2、从配置文件读取binder devices 信息 到device_names
		- 3、初始化binder设备（为binder驱动分配内存，设备配置初始化，将binder设备放到设备链表）
	- ### 2、通过binder_open()，打开binder设备（肯定binder通信的时候由client或者服务端打开这个驱动）
	  collapsed:: true
		- 1、创建[[binder_proc]]对象
			- （存储进程信息用，这样binder可以通过这个结构体跟踪管理每个进程的状态）
		- 2、将当前调用binder驱动的进程信息保存到binder_proc
		- 3、将proc对象放到filp这个文件指针的private_data，这样binder驱动下次通过filp就能找到这个proc了
			- ```c
			  传入的参数是filp
			  filp.private_data = proc
			  ```
		- 4、添加到binder_proc链表中
	- ### 3、通过binder_mmap()，在内核分配一块内存，用于存放数据
	  collapsed:: true
		- 1、通过用户空间的虚拟内存大小 ——>分配一块内核的虚拟内存
		  collapsed:: true
			- 【保证内核虚拟内存 和 用户空间的虚拟内存大小一致】
		- 2、分配了一块物理内存 4kb
		  collapsed:: true
			- 4k够用吗，为啥只分配4kb?
			  collapsed:: true
				- 因为现在还没开始通信，等实际使用的时候再添加
		- 3、把这块物理内存分别映射到  用户空间的虚拟内存和内核的虚拟内存
	- ### 4、binder_ioctl（）根据传入的命令 BINDER_WRITE_READ，进行读写操作
	  collapsed:: true
		- 流程是native层调用 ioctl(BINDER_WRITE_READ),进入内核层binder_ioctl 根据传入的命令 读写操作
		- binder_thread_write: 内核写操作
		- binder_thread_read：内核读操作