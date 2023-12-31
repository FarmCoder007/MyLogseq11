- ## 特点
	- 既能保证可见性，又能保证原子性
- ## 底层实现原理
	- Synchronized在JVM里的实现都是基于进入和退出Monitor对象来实现方法同步和代码块同步，虽然具体实现细节不一样，但是都可以通过成对的MonitorEnter和MonitorExit指令来实现。
	- ## 对同步块
		- MonitorEnter指令插入在同步代码块的开始位置，当代码执行到该指令时，[[#red]]==**将会尝试获取该对象Monitor的所有权，即尝试获得该对象的锁，**==
		- 而monitorExit指令则插入在方法结束处和异常处，JVM保证每个MonitorEnter必须有对应的MonitorExit。
		- ## 字节码
		  collapsed:: true
			- ![image.png](../assets/image_1690171612860_0.png)
	- ## 对同步方法，
	  collapsed:: true
		- 从同步方法反编译的结果来看，方法的同步并没有通过指令monitorenter和monitorexit来实现，相对于普通方法，其常量池中多了ACC_SYNCHRONIZED标示符。但是底层还是上边那两个指令
		- ![image.png](../assets/image_1690173055470_0.png)
	- JVM就是根据该标示符来实现方法的同步的：当方法被调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。
- ## synchronized使用的锁是存放在Java对象头里面，
  collapsed:: true
	- ![image.png](../assets/image_1690177489498_0.png)
	-
	- 具体位置是[[#red]]==**对象头里面的MarkWord**==，MarkWord里默认数据是存储对象的HashCode等信息，
	-
	- 但是会随着对象的运行改变而发生变化，不同的锁状态对应着不同的记录存储方式
	- ![image.png](../assets/image_1690177428258_0.png)
- ## [[synchronized 做的优化]]
- # 面试题
	- ## [[synchronized的实现原理+优化]]