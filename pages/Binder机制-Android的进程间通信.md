- # 一、什么是Binder机制
  title:: Binder机制-Android的进程间通信
	- 1、是Android进程间通信机制
	  collapsed:: true
		- Binder框架
			- ![image.png](../assets/image_1688269631168_0.png){:height 657, :width 718}
	- 2、内核层有一个[[Binder驱动]]设备
	- 3、Binder.java 是实现了Ibinder接口-实现了跨进程能力
- # 二、什么时候用到多进程？
  collapsed:: true
	- ## 自己创建的进程场景
		- webView 视频播放 音乐 大图浏览 推送
	- ## 系统服务场景
		- 打电话 闹钟 AMS
- # 三、多进程优点？
  collapsed:: true
	- 1、开辟更多内存，为一个app，一个进程分配的内存是有限的，多开一个进程，开辟的内存就会翻倍
	- 2、风险隔离：危险动作放在单独进程里，崩溃也不会影响主进程
- # 四、进程间通信为了什么？目的
  collapsed:: true
	- A进程 和 B进程之间通信是为了
		- A进程 调用 B进程的方法
		- A进程拿到B进程的数据
		- A进程传递数据给B进程
	- **==传递数据就需要拷贝==**
- # 五、[[Android中的Binder有什么优势？字节面试]]
- # 六、虚拟内存和物理内存的理解
  collapsed:: true
	- 虚拟内存是虚拟的
	- 物理内存是实际存在的
		- 内存条 磁盘
	-
	- 买房场景
		- 虚拟内存类似买房子的沙盘，是个假的不是真实的房子，看房选房时操作这个
		- 物理内存相当于真正的楼房
	- 或者说地球和地球仪的关系
		- 虚拟内存是地球仪，可以实际操作的
		- 物理内存是地球：放东西还是放在地球上
		-
- # 六、进程间怎么通信的？
  collapsed:: true
	- 进程间，用户空间内存是隔离的，内核空间是共享的。  不能直接拿到类 去new对象
	- ![image.png](../assets/image_1688272361964_0.png)
	- ## 传统IPC2次拷贝形象比喻
	  collapsed:: true
		- ![image.png](../assets/image_1688272719853_0.png)
		- ![image.png](../assets/image_1688272773413_0.png)
	- ## Bander的一次拷贝：[[Binder怎么做到一次拷贝的？腾讯面试]]
- # 七、[[Binder怎么做到一次拷贝的？腾讯面试]]
- # 八、[[Mmap原理讲解=腾讯面试]]
- # 九、Binder涉及到的类
  collapsed:: true
	- ![image.png](../assets/image_1688285372324_0.png)
- # 十、 [[Binder驱动]]内核层
- # 十一、[[Binder的Jni方法注册流程，JNI层代码]]
- # 十二、[[为什么Intent传输不能大于1M？]]
- # 十三、[[ServiceManager启动和获取]]框架的native层