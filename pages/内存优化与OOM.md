# 一、内存管理基础
collapsed:: true
	- ## 1、[[查看Android设备对APP的内存限制]]
	- ## 2、内存指标概念
	  collapsed:: true
		- ![image.png](../assets/image_1692754515271_0.png)
	- ## 3、内存分配与回收机制
		- [[JVM运行时数据区详解]]
		- [[对象与垃圾回收机制]]
	- ## 4、Android低内存杀进程机制
		- Anroid基于进程中运行的组件及其状态规定了默认的五个回收优先级：
			- ![image.png](../assets/image_1692754910999_0.png)
		- 最顶层优先回收
		- Empty process(空进程)
		  Background process(后台进程)
		  Service process(服务进程)
		  Visible process(可见进程)
		  Foreground process(前台进程)
		- 系统需要进行内存回收时最先回收空进程,然后是后台进程，以此类推最后才会回收前台进程（一般情况下前台进程就是与用户交互的进程了,如果连前台进程都需要回收那么此时系统几乎不可用了）。
		- ![image.png](../assets/image_1692755205569_0.png)
		- ActivityManagerService 会对所有进程进行评分（存放在变量adj中），然后再讲这个评分更新到内
		  核，由内核去完成真正的内存回收( lowmemorykiller , Oom_killer )。这里只是大概的流程，中间过
		  程还是很复杂的
		- [[maxAdj]]
		- 查看应用当前oom_adj值为多少：
			- ```java
			  car /proc/进程id/oom_adj
			  ```
- # 二、[[内存三大问题]]
- # 三、Android 内存分析命令介绍
  collapsed:: true
	- ## 常用的内存调优分析命令：
		- 1. [[dumpsys meminfo]]
		- 2. [[procrank]]
		- 3. [[cat /proc/meminfo]]
		- 4. [[free]]
		- 5. [[showmap]]
		- 6. [[vmstat]]
	- 总结
		- 1. dumpsys meminfo 适用场景： 查看进程的oom adj，或者dalvik/native等区域内存情况，或者某
		  个进程或apk的内存情况，功能非常强大；
		- 2. procrank 适用场景： 查看进程的VSS/RSS/PSS/USS各个内存指标；
		- 3. cat /proc/meminfo 适用场景： 查看系统的详尽内存信息，包含内核情况；
		- 4. free 适用场景： 只查看系统的可用内存；
		- 5. showmap 适用场景： 查看进程的虚拟地址空间的内存分配情况；
- # 四、内存泄漏分析工具
	- ## 1、[[MAT-Memory Analyzer Tool]]
	- ## 2、[Android Studio Memory-profiler](https://developer.android.com/studio/profile/memory-profiler#performance)
	- ## 3、[[LeakCanary]]
- # 五、[[Android内存泄漏常见场景以及解决方案]]
- # 六、[[Bitmap优化]]
- # 面试
	- ## [[内存优化：OOM+内存泄漏-面试]]