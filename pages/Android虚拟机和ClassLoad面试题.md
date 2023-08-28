# Android虚拟机面试题
collapsed:: true
	- # 1、[[Dalvik虚拟机和JVM虚拟机区别]]
	- # 2、什么是寄存器？
		- ## [[寄存器]]
	- # 3、[[那什么是基于栈的虚拟机，什么又是基于寄存器的虚拟机？]]
	- # 4、[[ART和Dalvik区别]]
- # 类加载器面试题
	- # [[类的生命周期]]
	- # 5、ClassLoader的分类
	  collapsed:: true
		- ## BootClassLoader
		  collapsed:: true
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
		  collapsed:: true
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
	- # 7、热修复的原理？
	  collapsed:: true
		- 需要讲述类加载的完整流程 + 热修复的插入点
		- 根据 [[双亲委托+类加载流程-面试]]类加载完整流程，
		- 1、获取当前应用的PathClassLoader
		- 2、反射获取到DexPathList，修改其中的dexElements
			- 1、把补丁包patch.dex 转换为 Element[]数组，插入到原 pathList 的dexElements最前边
	- # 8、[[双亲委托机制好处]]
	- # 9、ClassLoader使用场景
	  collapsed:: true
		- 1、插件化，加载插件apk中的类
		- 2、热修复，加载补丁.dex中的类