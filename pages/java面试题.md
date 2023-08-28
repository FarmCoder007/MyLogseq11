### 1、java为什么具有跨平台性？
	- 1、java程序[[#red]]==**基于JVM虚拟机**==运行的，而不是直接运行在操作系统上，JVM虚拟机在不同操作系统上实现了统一接口，负责java字节码的解释执行。
	- 2、java会编译成class字节码，与系统平台无关中间代码，可以在任何安装了JVM的平台上运行
- ## [[整数进制]]
- ## [[计算机存储单位和内存计算]]
-
- ## 1、final、finally、finallize区别
	- final不可变修饰符
	- finally，try catch finally 异常处理最后执行的方法,用于资源关闭等
	- finalize：
		- 垃圾回收器确定该对象没有更多引用，由对象的垃圾回收器调用此方法
- ## 2、throw和throws区别
	- 1，throws使用在函数上。
	     throw使用在函数内。
	- 2，throws抛出的是异常类，可以抛出多个，用逗号隔开。
	     throw抛出的是异常对象。
- ## 3、if else 和 Switch区别
  collapsed:: true
	- ### 一、if-else
		- 只是单纯地一个接一个比较；[[#red]]==**if...else每个条件都计算一遍；**==
	- ### 二、switch
		- 使用了Binary Tree算法；[[#red]]==**绝大部分情况下switch会快一点，除非是if-else的第一个条件就为true**==
		- 编译器编译switch与编译if...else...不同。[[#red]]==**不管有多少case，都直接跳转，不需逐个比较查询**==；switch只计算一次值，然后都是test , jmp,
	- 有很多else if的时候，用switch case比较清晰
	-
	- switch使用查找表的方式决定了case的条件必须是一个连续的常量。而if-else则可以灵活的多。
	- ### 三、总结
		- 当只有分支比较少的时候，if效率比switch高（因为switch有跳转表）
		- 分支比较多，那当然是switch
	- ### 四、[[switch case语句]]
	-
- # 4、[[访问修饰符]]
- # 5、java的String和android的String有什么区别？
	- android确实是通过native方法对常用的api进行了优化吧