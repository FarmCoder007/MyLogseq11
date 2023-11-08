# Android虚拟机面试题
	- # 1、[[Dalvik虚拟机和JVM虚拟机区别]]#card
		- ## 1、执行文件不同
			- Dalvik虚拟机运行Dex（多个字节码压缩产物）
			- JVM运行的是字节码
		- ## 2、执行的指令集不一样
			- Dalvik的指令集基于寄存器的
			- JVM基于栈的（指的是虚拟机栈中的操作数栈）
	- # 2、什么是寄存器？#card
		- ## 1、真实寄存器
			- 1、[[#red]]==**寄存器是CPU的组成部分**==，==**有限存储容量**==的[[#red]]==**高速存储部件**==，是物理的
		- ## 2、Android虚拟机的虚拟寄存器
			- 1、只是虚拟寄存器，==**指令交互上模拟真实寄存器而已**==。虚拟寄存器就是相当于一块内存。
			- 2、相当于把栈式虚拟机==**局部变量表 和 操作数栈 合并成一个虚拟寄存器**==，只用来存储，运算直接交给cpu，少了把数据从局部变量表 和 操作数栈 中移动过程
	- # 3、[[那什么是基于栈的虚拟机，什么又是基于寄存器的虚拟机？]]#card
		- ## 一、栈式虚拟机
			- 1、==**每一个运行的线程都有一个独立的虚拟机栈**==[[#red]]==**每一个方法调用会多一个栈帧**==。
			- 2、然后**==执行指令主要涉及局部变量表 和 操作数栈 来回加载==**
			- 3、栈式虚拟机[[#red]]==**不管进行何种操作都要通过操作数栈来进行**==，即使是数据传递这种简单的操作
		- ## 二、寄存器式虚拟机
			- 1、[[#red]]==**没有操作数栈，但是有很多虚拟寄存器**==，[[#red]]==**相当于把 栈式虚拟机 局部变量表 和 操作数栈 合并成一个区域**==，只用来存储，运算直接交给cpu
			- 2、少了把数据从局部变量表 和 操作数栈 中移动
		- ## 优缺点
			- ## 栈式虚拟机
				- 优点：和操作数栈对接，虚拟机可以无视具体物理架构，可移植性高
				- 缺点：速度稍慢,因为无论什么操作都要通过操作数栈这一结构
			- ## 寄存器式虚拟机
				- 优点：所以速度快一些。
				- 缺点：由于寄存器的指令集是与硬件的架构紧密关联的，可移植性差
	- # 4、[[ART和Dalvik区别]]小红书#card
		- ## Dalvik
			- 1、运行==**dex文件**==
			- 2、使用解释执行+JIT即时编译
		- ##  ART-Android Runtime
			- 1、运行的==**机器码**==
			- 2、Android N 变成了 解析执行+JIT + 空闲时 AOT预先编译
- # 类加载器面试题
	- # [[类的生命周期]]
	- # 5、ClassLoader的分类#Card
		- ## BootClassLoader
			- 用于加载Android[[#red]]==**Framework层class文件**==
		- ## PathClassLoader（平时我们操作的是这个）
		  collapsed:: true
			- 用于[[#red]]==**Android应用程序类加载器**==。[[#red]]==**也可以加载指定的dex，以及jar、zip、apk中的classes.dex**==
			- > 很多博客里说PathClassLoader只能加载已安装的apk的dex，其实这说的应该是在dalvik虚拟机上。但现在一般不用关心dalvik了。
			- 源码
				- ```java
				  public class PathClassLoader extends BaseDexClassLoader {
				    public PathClassLoader(String dexPath, ClassLoader parent) {
				      super(dexPath, null, null, parent);
				    }
				    public PathClassLoader(String dexPath, String librarySearchPath, ClassLoader parent){
				    		super(dexPath, null, librarySearchPath, parent);
				    }
				  }
				  ```
		- ## DexClassLoader
			- [[#red]]==**用于加载指定的dex，以及jar、zip、apk中的classes.dex**==
			- 源码
				- ```java
				  public class DexClassLoader extends BaseDexClassLoader {
				      public DexClassLoader(String dexPath, String optimizedDirectory,
				          String librarySearchPath, ClassLoader parent) {
				          super(dexPath, new File(optimizedDirectory), librarySearchPath, parent);
				      }
				  }
				  ```
	- # 6、什么是双亲委托机制？类加载机制的流程？
		- ## [[双亲委托+类加载流程-面试]]
	- # 7、热修复的原理？#Card
		- 需要讲述类加载的完整流程 + 热修复的插入点
		- 根据 [[双亲委托+类加载流程-面试]]类加载完整流程，
		- 1、获取当前==**应用的PathClassLoader**==
		- 2、反射获取到[[#red]]==**DexPathList**==，修改其中的[[#red]]==**dexElements**==
			- 1、把补丁包patch.dex 转换为 Element[]数组，插入到原 pathList 的dexElements最前边
	- # 8、[[双亲委托机制好处]]#Card
		- 1、避免重复加载
		- 2、安全
	- # 9、ClassLoader使用场景#Card
		- 1、插件化，加载插件apk中的类
		- 2、热修复，加载补丁.dex中的类