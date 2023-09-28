title:: trace.txt文件解读

- ## 1、人为的收集trace.txt的命令
	- adb shell kill -3 888 //可指定进程pid
	- 执行完该命令后traces信息的结果保存到文件/data/anr/traces.txt
- ## 2、trace文件解读
  collapsed:: true
	- ```java
	  ----- pid 888 at 2016-11-11 22:22:22 -----
	  Cmd line: system_server
	  ABI: arm
	  Build type: optimized
	  Zygote loaded classes=4113 post zygote classes=3239
	  Intern table: 57550 strong; 9315 weak
	  JNI: CheckJNI is off; globals=2418 (plus 115 weak)
	  Libraries: /system/lib/libandroid.so /system/lib/libandroid_servers.so
	  /system/lib/libaudioeffect_jni.so /system/lib/libcompiler_rt.so /system/lib/libjavacrypto.so
	  /system/lib/libjnigraphics.so /system/lib/libmedia_jni.so /system/lib/librs_jni.so
	  /system/lib/libsechook.so /system/lib/libshell_jni.so /system/lib/libsoundpool.so
	  /system/lib/libwebviewchromium_loader.so /system/lib/libwifi-service.so
	  /vendor/lib/libalarmservice_jni.so /vendor/lib/liblocationservice.so libjavacore.so (16)
	  //已分配堆内存大小40MB，其中29M已用，总分配207772个对象
	  Heap: 27% free, 29MB/40MB; 307772 objects
	  ... //省略GC相关信息
	  //当前进程总99个线程
	  DALVIK THREADS (99):
	  //主线程调用栈
	  "main" prio=5 tid=1 Native
	  | group="main" sCount=1 dsCount=0 obj=0x75bd9fb0 self=0x5573d4f770
	  | sysTid=12078 nice=-2 cgrp=default sched=0/0 handle=0x7fa75fafe8
	  | state=S schedstat=( 5907843636 827600677 5112 ) utm=453 stm=137 core=0 HZ=100
	  | stack=0x7fd64ef000-0x7fd64f1000 stackSize=8MB
	  | held mutexes=
	  //内核栈
	  kernel: __switch_to+0x70/0x7c
	    kernel: SyS_epoll_wait+0x2a0/0x324
	  kernel: SyS_epoll_pwait+0xa4/0x120
	  kernel: cpu_switch_to+0x48/0x4c
	  native: #00 pc 0000000000069be4 /system/lib64/libc.so (__epoll_pwait+8)
	  native: #01 pc 000000000001cca4 /system/lib64/libc.so (epoll_pwait+32)
	  native: #02 pc 000000000001ad74 /system/lib64/libutils.so (_ZN7android6Looper9pollInnerEi+144)
	  native: #03 pc 000000000001b154 /system/lib64/libutils.so
	  (_ZN7android6Looper8pollOnceEiPiS1_PPv+80)
	  native: #04 pc 00000000000d4bc0 /system/lib64/libandroid_runtime.so
	  (_ZN7android18NativeMessageQueue8pollOnceEP7_JNIEnvP8_jobjecti+48)
	  native: #05 pc 000000000000082c /data/dalvik-cache/arm64/system@framework@boot.oat
	  (Java_android_os_MessageQueue_nativePollOnce__JI+144)
	  at android.os.MessageQueue.nativePollOnce(Native method)
	  at android.os.MessageQueue.next(MessageQueue.java:323)
	  at android.os.Looper.loop(Looper.java:135)
	  at com.android.server.SystemServer.run(SystemServer.java:290)
	  at com.android.server.SystemServer.main(SystemServer.java:175)
	  at java.lang.reflect.Method.invoke!(Native method)
	  at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:738)
	  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:628)
	  "Binder_1" prio=5 tid=8 Native
	  | group="main" sCount=1 dsCount=0 obj=0x12c610a0 self=0x5573e5c750
	  | sysTid=12092 nice=0 cgrp=default sched=0/0 handle=0x7fa2743450
	  | state=S schedstat=( 796240075 863170759 3586 ) utm=50 stm=29 core=1 HZ=100
	  | stack=0x7fa2647000-0x7fa2649000 stackSize=1013KB
	  | held mutexes=
	  kernel: __switch_to+0x70/0x7c
	  kernel: binder_thread_read+0xd78/0xeb0
	  kernel: binder_ioctl_write_read+0x178/0x24c
	  kernel: binder_ioctl+0x2b0/0x5e0
	  kernel: do_vfs_ioctl+0x4a4/0x578
	  kernel: SyS_ioctl+0x5c/0x88
	  kernel: cpu_switch_to+0x48/0x4c
	  native: #00 pc 0000000000069cd0 /system/lib64/libc.so (__ioctl+4)
	  native: #01 pc 0000000000073cf4 /system/lib64/libc.so (ioctl+100)
	  native: #02 pc 000000000002d6e8 /system/lib64/libbinder.so
	  (_ZN7android14IPCThreadState14talkWithDriverEb+164)
	  native: #03 pc 000000000002df3c /system/lib64/libbinder.so
	  (_ZN7android14IPCThreadState20getAndExecuteCommandEv+24)
	  native: #04 pc 000000000002e114 /system/lib64/libbinder.so
	  (_ZN7android14IPCThreadState14joinThreadPoolEb+124)
	  native: #05 pc 0000000000036c38 /system/lib64/libbinder.so (???)
	  native: #06 pc 000000000001579c /system/lib64/libutils.so
	  (_ZN7android6Thread11_threadLoopEPv+208)
	  native: #07 pc 0000000000090598 /system/lib64/libandroid_runtime.so
	  (_ZN7android14AndroidRuntime15javaThreadShellEPv+96)
	  native: #08 pc 0000000000014fec /system/lib64/libutils.so (???)
	  native: #09 pc 0000000000067754 /system/lib64/libc.so (_ZL15__pthread_startPv+52)
	  native: #10 pc 000000000001c644 /system/lib64/libc.so (__start_thread+16)
	  (no managed stack frames)
	  ... //此处省略剩余的N个线程.
	  ```
- ## 3、trace参数解读
	- ```java
	  "Binder_1" prio=5 tid=8 Native
	  | group="main" sCount=1 dsCount=0 obj=0x12c610a0 self=0x5573e5c750
	  | sysTid=12092 nice=0 cgrp=default sched=0/0 handle=0x7fa2743450
	  | state=S schedstat=( 796240075 863170759 3586 ) utm=50 stm=29 core=1 HZ=100
	  | stack=0x7fa2647000-0x7fa2649000 stackSize=1013KB
	  | held mutexes=
	  ```
	- ## 第0行:
	  collapsed:: true
		- 线程名: Binder_1（如有daemon则代表守护线程)
		- prio: 线程优先级
		- tid: 线程内部id
		- 线程状态: NATIVE
	- ##  第1行
	  collapsed:: true
		- group: 线程所属的线程组
		- sCount: 线程挂起次数
		- dsCount: 用于调试的线程挂起次数
		- obj: 当前线程关联的java线程对象
		- self: 当前线程地址
	- ## 第2行：
		- sysTid：线程真正意义上的tid
		- nice: 调度有优先级
		- cgrp: 进程所属的进程调度组
		- sched: 调度策略
		- handle: 函数处理地址
	- ## 第3行
		- state: 线程状态
		- schedstat: CPU调度时间统计
		- utm/stm: 用户态/内核态的CPU时间(单位是jiffies)
		- core: 该线程的最后运行所在核
		- HZ: 时钟频率
	- ## 第4行：
		- stack：线程栈的地址区间
		- stackSize：栈的大小
	- ## 第5行：
		- mutex: 所持有mutex类型，有独占锁exclusive和共享锁shared两类
	- ## schedstat含义说明：
		- nice值越小则优先级越高。此处nice=-2, 可见优先级还是比较高的;
		- schedstat括号中的3个数字依次是Running、Runable、Switch，紧接着的是utm和stm
			- Running时间：CPU运行的时间，单位ns
			- Runable时间：RQ队列的等待时间，单位ns
			- Switch次数：CPU调度切换次数
			- utm: 该线程在用户态所执行的时间，单位是jiffies，jiffies定义为sysconf(_SC_CLK_TCK)，默认等
			  于10ms
			- stm: 该线程在内核态所执行的时间，单位是jiffies，默认等于10ms
		- 可见，该线程Running=186667489018ns,也约等于186667ms。在CPU运行时间包括用户态(utm)和内核态(stm)。 utm + stm = (12112 + 6554) ×10 ms = 186666ms。
		- 结论：utm + stm = schedstat第一个参数值