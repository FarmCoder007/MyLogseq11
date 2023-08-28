## 一、 查看events_log
collapsed:: true
	- 查看mobilelog文件夹下的events_log,从日志中搜索关键字：`am_anr`，找到出现ANR的时间点、进程PID、ANR类型。
	  
	  如日志：
	  
	  ```text
	  07-20 15:36:36.472  1000  1520  1597 I am_anr  : [0,1480,com.xxxx.moblie,952680005,Input dispatching timed out (AppWindowToken{da8f666 token=Token{5501f51 ActivityRecord{15c5c78 u0 com.xxxx.moblie/.ui.MainActivity t3862}}}, Waiting because no window has focus but there is a focused application that may eventually add a window when it finishes starting up.)]
	  ```
	  
	  从上面的log我们可以看出： 应用`com.xxxx.moblie` 在`07-20 15:36:36.472`时间，发生了一次`KeyDispatchTimeout`类型的ANR，它的进程号是`1480`. 把关键的信息整理一下：
	  **ANR时间**：07-20 15:36:36.472
	  **进程pid**：1480
	  **进程名**：com.xxxx.moblie
	  **ANR类型**：KeyDispatchTimeout
	  
	  我们已经知道了发生`KeyDispatchTimeout`的ANR是因为 `input事件在5秒内没有处理完成`。那么在这个时间`07-20 15:36:36.472` 的前5秒，也就是（`15:36:30 ~15:36:31`）时间段左右程序到底做了什么事情？这个简单，因为我们已经知道pid了，再搜索一下`pid = 1480`的日志.这些日志表示该进程所运行的轨迹，关键的日志如下：
	  
	  ```text
	  07-20 15:36:29.749 10102  1480  1737 D moblie-Application: [Thread:17329] receive an intent from server, action=com.ttt.push.RECEIVE_MESSAGE
	  07-20 15:36:30.136 10102  1480  1737 D moblie-Application: receiving an empty message, drop
	  07-20 15:36:35.791 10102  1480  1766 I Adreno  : QUALCOMM build                   : 9c9b012, I92eb381bc9
	  07-20 15:36:35.791 10102  1480  1766 I Adreno  : Build Date                       : 12/31/17
	  07-20 15:36:35.791 10102  1480  1766 I Adreno  : OpenGL ES Shader Compiler Version: EV031.22.00.01
	  07-20 15:36:35.791 10102  1480  1766 I Adreno  : Local Branch                     : 
	  07-20 15:36:35.791 10102  1480  1766 I Adreno  : Remote Branch                    : refs/tags/AU_LINUX_ANDROID_LA.UM.6.4.R1.08.00.00.309.049
	  07-20 15:36:35.791 10102  1480  1766 I Adreno  : Remote Branch                    : NONE
	  07-20 15:36:35.791 10102  1480  1766 I Adreno  : Reconstruct Branch               : NOTHING
	  07-20 15:36:35.826 10102  1480  1766 I vndksupport: sphal namespace is not configured for this process. Loading /vendor/lib64/hw/gralloc.msm8998.so from the current namespace instead.
	  07-20 15:36:36.682 10102  1480  1480 W ViewRootImpl[MainActivity]: Cancelling event due to no window focus: KeyEvent { action=ACTION_UP, keyCode=KEYCODE_PERIOD, scanCode=0, metaState=0, flags=0x28, repeatCount=0, eventTime=16099429, downTime=16099429, deviceId=-1, source=0x101 }
	  ```
	  
	  从上面我们可以知道，在时间 07-20 15:36:29.749 程序收到了一个action消息。
	  
	  ```text
	  07-20 15:36:29.749 10102  1480  1737 D moblie-Application: [Thread:17329] receive an intent from server, action=com.ttt.push.RECEIVE_MESSAGE。
	  ```
	  
	  原来是应用`com.xxxx.moblie` 收到了一个推送消息（`com.ttt.push.RECEIVE_MESSAGE`）导致了阻塞，我们再串联一下目前所获取到的信息：在时间`07-20 15:36:29.749` 应用`com.xxxx.moblie` 收到了一下推送信息`action=com.ttt.push.RECEIVE_MESSAGE`发生阻塞，5秒后发生了`KeyDispatchTimeout的ANR`。
	  
	  虽然知道了是怎么开始的，但是具体原因还没有找到，是不是当时CPU很紧张、各路APP再抢占资源？ 我们再看看CPU的信息,。搜索关键字关键字： `ANR IN`
	  
	  ```text
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager: ANR in com.xxxx.moblie (com.xxxx.moblie/.ui.MainActivity) (进程名)
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager: PID: 1480 (进程pid)
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager: Reason: Input dispatching timed out (AppWindowToken{da8f666 token=Token{5501f51 ActivityRecord{15c5c78 u0 com.xxxx.moblie/.ui.MainActivity t3862}}}, Waiting because no window has focus but there is a focused application that may eventually add a window when it finishes starting up.)
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager: Load: 0.0 / 0.0 / 0.0 (Load表明是1分钟,5分钟,15分钟CPU的负载)
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager: CPU usage from 20ms to 20286ms later (2018-07-20 15:36:36.170 to 2018-07-20 15:36:56.436):
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   42% 6774/pressure: 41% user + 1.4% kernel / faults: 168 minor
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   34% 142/kswapd0: 0% user + 34% kernel
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   31% 1520/system_server: 13% user + 18% kernel / faults: 58724 minor 1585 major
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   13% 29901/com.ss.android.article.news: 7.7% user + 6% kernel / faults: 56007 minor 2446 major
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   13% 32638/com.android.quicksearchbox: 9.4% user + 3.8% kernel / faults: 48999 minor 1540 major
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   11% (CPU的使用率)1480/com.xxxx.moblie: 5.2%(用户态的使用率) user + (内核态的使用率) 6.3% kernel / faults: 76401 minor 2422 major
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   8.2% 21000/kworker/u16:12: 0% user + 8.2% kernel
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   0.8% 724/mtd: 0% user + 0.8% kernel / faults: 1561 minor 9 major
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   8% 29704/kworker/u16:8: 0% user + 8% kernel
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   7.9% 24391/kworker/u16:18: 0% user + 7.9% kernel
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   7.1% 30656/kworker/u16:14: 0% user + 7.1% kernel
	  07-20 15:36:58.711  1000  1520  1597 E ActivityManager:   7.1% 9998/kworker/u16:4: 0% user + 7.1% kernel
	  ```
	  
	  我已经在log 中标志了相关的含义。`com.xxxx.moblie` 占用了11%的CPU，其实这并不算多。现在的手机基本都是多核CPU。假如你的CPU是4核，那么上限是400%，以此类推。
	  
	  既然不是CPU负载的原因，那么到底是什么原因呢？ 这时就要看我们的终极大杀器——`traces.txt`。
- ## 二、 traces.txt 日志分析
  collapsed:: true
	- 当APP不响应、响应慢了、或者WatchDog的监视没有得到回应时，系统就会dump出一个`traces.txt`文件，存放在文件目录:`/data/anr/traces.txt`，通过traces文件,我们可以拿到线程名、堆栈信息、线程当前状态、binder call等信息。
	  通过adb命令拿到该文件：`adb pull /data/anr/traces.txt`
	  trace: Cmd line:com.xxxx.moblie
	  
	  ```text
	  "main" prio=5 tid=1 Runnable
	  | group="main" sCount=0 dsCount=0 obj=0x73bcc7d0 self=0x7f20814c00
	  | sysTid=20176 nice=-10 cgrp=default sched=0/0 handle=0x7f251349b0
	  | state=R schedstat=( 0 0 0 ) utm=12 stm=3 core=5 HZ=100
	  | stack=0x7fdb75e000-0x7fdb760000 stackSize=8MB
	  | held mutexes= "mutator lock"(shared held)
	  // java 堆栈调用信息,可以查看调用的关系，定位到具体位置
	  at ttt.push.InterceptorProxy.addMiuiApplication(InterceptorProxy.java:77)
	  at ttt.push.InterceptorProxy.create(InterceptorProxy.java:59)
	  at android.app.Activity.onCreate(Activity.java:1041)
	  at miui.app.Activity.onCreate(SourceFile:47)
	  at com.xxxx.moblie.ui.b.onCreate(SourceFile:172)
	  at com.xxxx.moblie.ui.MainActivity.onCreate(SourceFile:68)
	  at android.app.Activity.performCreate(Activity.java:7050)
	  at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1214)
	  at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2807)
	  at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2929)
	  at android.app.ActivityThread.-wrap11(ActivityThread.java:-1)
	  at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1618)
	  at android.os.Handler.dispatchMessage(Handler.java:105)
	  at android.os.Looper.loop(Looper.java:171)
	  at android.app.ActivityThread.main(ActivityThread.java:6699)
	  at java.lang.reflect.Method.invoke(Native method)
	  at com.android.internal.os.Zygote$MethodAndArgsCaller.run(Zygote.java:246)
	  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:783)
	  ```
	  
	  我详细解析一下`traces.txt`里面的一些字段，看看它到底能给我们提供什么信息.
	  **main**：main标识是主线程，如果是线程，那么命名成“Thread-X”的格式,x表示线程id,逐步递增。
	  **prio**：线程优先级,默认是5
	  **tid**：tid不是线程的id，是线程唯一标识ID
	  **group**：是线程组名称
	  **sCount**：该线程被挂起的次数
	  **dsCount**：是线程被调试器挂起的次数
	  **obj**：对象地址
	  **self**：该线程Native的地址
	  **sysTid**：是线程号(主线程的线程号和进程号相同)
	  **nice**：是线程的调度优先级
	  **sched**：分别标志了线程的调度策略和优先级
	  **cgrp**：调度归属组
	  **handle**：线程处理函数的地址。
	  **state**：是调度状态
	  **schedstat**：从 `/proc/[pid]/task/[tid]/schedstat`读出，三个值分别表示线程在cpu上执行的时间、线程的等待时间和线程执行的时间片长度，不支持这项信息的三个值都是0；
	  **utm**：是线程用户态下使用的时间值(单位是jiffies）
	  **stm**：是内核态下的调度时间值
	  **core**：是最后执行这个线程的cpu核的序号。
- Java的堆栈信息是我们最关心的，它能够定位到具体位置。从上面的traces,我们可以判断`ttt.push.InterceptorProxy.addMiuiApplicationInterceptorProxy.java:77` 导致了`com.xxxx.moblie`发生了ANR。这时候可以对着源码查看，找到出问题，并且解决它。
- 总结一下这分析流程：首先我们搜索`am_anr`，找到出现ANR的时间点、进程PID、ANR类型、然后再找搜索`PID`，找前5秒左右的日志。过滤ANR IN 查看CPU信息，接着查看`traces.txt`，找到java的堆栈信息定位代码位置，最后查看源码，分析与解决问题。这个过程基本能找到发生ANR的来龙去脉。
- ## [[ANR 案例整理]]
	-