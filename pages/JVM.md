# 一、JVM组成
	- ![jvm组成.png](../assets/jvm组成_1684819371991_0.png)
	- ## JVM包含两个子系统和两个组件。
		- ## 两个子系统:
			- ## [[ClassLoader]](类装载)：
				- 根据给定的全限定名类名(如：java.lang.Object)来装载class文件到Runtime data area中的method area。
			- ## Execution engine（执行引擎）：
				- 执行classes中的指令。
		- ## 两个组件:
			- ## Native Interface(本地接口)：
				- 与native libraries交互，是其它编程语言交互的接口。
			- ## Runtime data area：[[JVM运行时数据区详解]]
				- (运行时数据区域)这就是我们常说的JVM的内存。
- # 二、虚拟机与操作系统的关系
	- JVM只是一个翻译，JRE提供了基础类库，JDK提供了工具
	- 翻译：JVM下联操作系统，上连字节码、把字节码翻译生机器码
	- ![image.png](../assets/image_1689392850615_0.png)
	- 在了解JVM内存管理之前，我们先简单介绍一下虚拟机与操作系统的关系。Linux\Windows、MacOS等操作系统，它们识别（运行）的是机器码：010101… 而我们写的java代码实际上都是字符串，是不能直接运行在操作系统上的。必须编译成虚拟机识别的字节码（.class）,字节码也不能直接交给CPU执行，他必须要经过一个解释器，解释器是Java虚拟机执行引擎的一个组件，专门负责把每一条JVM指令解释成机器码，机器码就可以交给CPU执行了。可以说虚拟机就类似于一个“翻译软件”。
	- [[#red]]==**jdk默认的HotSpot，以及android的Dalvik、ART都是虚拟机**==，当然安卓中的虚拟机和Java虚拟机还是有区别的，比如[[#red]]==**java虚拟机都是基于栈的，android虚拟机是基于寄存器，**==后面再讲。
- # 三、jvm运行过程
	- 一个java文件从编码到执行需要经过下面几个阶段
	  1、编译阶段：首先.java文件经过javac编译成.class文件
	  2、加载阶段：然后.class文件经过类的加载器加载到JVM内存，即运行时数据区。
	  3、解释阶段：class字节码经过字节码解释器解释成操作系统可识别的指令码。
	  4、执行阶段：执行引擎向硬件设备发送指令码执行操作。
	- ![image.png](../assets/image_1684431191329_0.png){:height 781, :width 603}
- # 四、[[JVM常量池梳理]]
- # 五、[[内存可视化工具HSDB]]
- # 六、[[对象与垃圾回收机制]]
-
-
- # [[JVM虚拟机面试题]]