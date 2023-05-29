- # 一、什么是单例设计模式
  collapsed:: true
	- 确保某一个类在整个项目中只有一个实例，并且自行创建实例化对象，并向整个系统提供这个实例。
- # 二、创建流程
  collapsed:: true
	- 1、私有化所有的构造方法(不通过外部创建该类的对象)
	- 2、给外部提供一个获得该类对象的方法(类方法 static修饰,因为此时没有该类的对象，不能通过对象来调用此方法)
	- 3、必须创建一个私有的静态成员变量来保存当前唯一的对象(使用静态方法只能访问静态的变量)
	- 4.创建类的一个对象
- # 三、八种创建方式
	- ## 一、饿汉式（2种）
	  collapsed:: true
		- ## 实现原理
		  collapsed:: true
			- 这种方式[[#red]]==**基于classloder机制避免了多线程的同步问题**==，instance在类装载 时就实例化，在单例模式中大多数都是调用getInstance方法，但是导致类装载的原因有很多种，因此不能确定有其他的方式（或者其他的静态方法）导致类装载，这时候初始化instance就没有达到lazyloading的效果
		- ## 特点
		  collapsed:: true
			- 优点：
				- **简单直接**，实例在类加载时就被创建，保证了**线程安全**。
				- **性能较好**，因为实例在类加载时就被创建，**不需要进行额外的同步操作**。
			- 缺点：
				- 1、类加载的时候就完成了实例化，没有懒加载效果，可能会造成资源浪费
		- ## 结论：
		  collapsed:: true
			- 线程安全，但是可能造成内存浪费
		- ## 写法
			- ## 静态常量写法
				- ## java
					- ```java
					  public class Singleton {
					      // 1、构造函数私有化
					      private Singleton() {}
					      // 2、内部创建对象实例
					      private static Singleton instance =  new Singleton();	
					      
					      // 3、对外提供公有方法，获取实例
					      public static Singleton getInstance() {
					          return instance;
					      }
					  }
					  ```
				- ## kotlin
					- object修饰的类，既是[单例]的实例，又是类名，又是对象
					- ```kotlin
					  
					  object BaseSingleton {
					   
					  }
					  ```
			- ## 静态代码块写法
			  collapsed:: true
				- ## java
					- ```java
					  public class Singleton {
					      // 1、构造函数私有化
					      private Singleton() {}
					      // 2、内部创建对象实例
					      private static Singleton instance;
					      // 3、静态代码块中创建单例
					      static {
					        instance =  new Singleton();
					      }
					      
					      // 3、对外提供公有方法，获取实例
					      public static Singleton getInstance() {
					          return instance;
					      }
					  }
					  ```
	- ## 二、懒汉式（4种）
		- ## 懒汉式（线程不安全）【不能用】
		  collapsed:: true
			- ## 特点
			  collapsed:: true
				- ## 优点：
				  collapsed:: true
					- 起到懒加载效果
				- ## 缺点：
				  collapsed:: true
					- 线程不安全
					- 如果在多线程下，一个线程进入了if (singleton ==null)判断语句块，还未来得及往下执行，另一个线程也通过了这个判断语句，这时便会产生多个实例。所以在多线程环境下不可使用这种方式
			- ## 结论：**不要使用这种方式.**
			- ## java
			  collapsed:: true
				- ```java
				  public class Singleton {
				      private static Singleton instance;
				      
				      private Singleton() {}
				      
				      public static Singleton getInstance() {
				          if (instance == null) {
				              instance = new Singleton();
				          }
				          return instance;
				      }
				  }
				  ```
			- ## kotlin
				- ```kotlin
				  /**
				   * Description:懒汉式单例
				   */
				  class LazyLoadSingleton private constructor() {
				      /**
				       * companion object相当于Java中的 public static 类似于静态代码块
				       */
				      companion object {
				          //LazyThreadSafetyMode.NONE线程非安全  ——  对比与双重检测模式 LazyThreadSafetyMode.SYNCHRONIZED
				          val INSTANCE by lazy(LazyThreadSafetyMode.NONE) {
				              LazyLoadSingleton();
				          }
				      }
				  }
				  ```
		- ## 懒汉式（线程安全，synchronized+静态方法）【不推荐使用】
		  collapsed:: true
			- ## 特点
				- 优点：实现简单，实现了懒加载。
				- 缺点：方法进行线程同步效率低，性能差
			- ## 结论
				- 不推荐使用
			- ## 代码示例
				- ## java
					- ```java
					  public class Singleton {
					      private static Singleton instance;
					      
					      private Singleton() {}
					      
					      public static synchronized Singleton getInstance() {
					          if (instance == null) {
					              instance = new Singleton();
					          }
					          return instance;
					      }
					  }
					  ```
				- ## kotlin
					- ```kotlin
					  /**
					   * Description: 同步锁式单例
					   * 优点：线程安全
					   * 缺点：每次获取实例都会加锁
					   */
					  class LazySynchronizedSingleton private constructor() {
					      /**
					       * companion object相当于Java中的 public static 类似于静态代码块
					       */
					      companion object {
					          //kotlin中变量后面有？表示该变量可以为null
					          private var INSTANCE : LazySynchronizedSingleton? = null
					   
					          /**
					           * 在kotlin中通过注解方式加锁
					           * !!表示非空判断
					           */
					          @Synchronized
					          fun getInstance(): LazySynchronizedSingleton {
					              if (INSTANCE == null) {
					                  INSTANCE = LazySynchronizedSingleton()
					              }
					              return INSTANCE!!
					          }
					      }
					  }
					  ```
		- ## 懒汉式（线程安全，同步代码块）【本质线程不安全，不能用】
		  collapsed:: true
			- ## 特点
			  collapsed:: true
				- 本意是想对上个实现方式的改进，因为前面同步方法效率太低，改为同步产生实例化的的代码块
				- 但是[[#red]]==**这种同步并不能起到线程同步的作用**==。跟线程不安全版本遇到的情形一致，假如一个线程进入了if (singleton ==null)判断语句块，还未来得及往下执行，另一个线程也通过了这个判断语句，这时便会产生多个实例
				- ## 缺点
				  collapsed:: true
					- 并不能解决同步问题
			- ## 结论
			  collapsed:: true
				- 其实这种方式是不安全的，并不能真正实现单例，因为如果，有两个线程都进入到singleton == null为空的判断为true中，那么这两个线程都会创建实例的
				- 需要双重校验
			- ## 代码示例
			  collapsed:: true
				- ## java
					- ```java
					  public class Singleton {
					      private static Singleton instance;
					      
					      private Singleton() {}
					      
					      public static Singleton getInstance() {
					          if (instance == null) {
					              synchronized (Singleton.class){
					                	instance = new Singleton();
					              }
					          }
					          return instance;
					      }
					  }
					  ```
				- ## kotlin
					- ```kotlin
					  ```
		- ## [[#red]]==**懒汉式（双重检查，线程安全）【推荐】**==
		  collapsed:: true
			- ## 特点
				- ## 优点：
				  collapsed:: true
					- - **延迟加载**，只有在需要时才会实例化对象。
					- - **效率高**：在多线程环境下保持了较好的性能。
					- - 线程安全
				- ## 缺点：
					- 实现较复杂，需要使用 volatile 关键字和双重检查来保证线程安全。
			- ## 代码示例
			  collapsed:: true
				- ## java
					- ```java
					  public class Singleton {
					      private static volatile Singleton instance;
					      
					      private Singleton() {}
					      
					      public static Singleton getInstance() {
					          if (instance == null) {
					              synchronized (Singleton.class) {
					                  if (instance == null) {
					                      instance = new Singleton();
					                  }
					              }
					          }
					          return instance;
					      }
					  }
					  ```
				- ## kotlin
					- ```kotlin 
					  class SingletonDemo private constructor() {	
					      companion object {
					          val instance : SingletonDemo by lazy (mode = LazyThreadSafetyMode.SYNCHRONIZED) { SingletonDemo() }
					      }	
					  }
					  ```
	- ## 三、[[#red]]==**静态内部类（1种）【推荐】**==
	  collapsed:: true
		- ## 原理
		  collapsed:: true
			- - 静态内部类方式在Singleton类被装载时**并不会立即实例化**，而是在需要实例化时，调用getInstance方法，才会装载singletonInstance类，从而完成Singleton的实例化。
		- ## 特点
		  collapsed:: true
			- ## 优点：
				- - 实现了延迟加载，只有在需要时才会实例化对象。
				- - 保证了线程安全。
				- - 通过静态内部类的方式，将实例的创建和初始化过程与外部类隔离开来，提高了代码的可读性和维护性。
			- ## 缺点：
				- 暂无
		- ## 代码示例
			- ## Java
			  collapsed:: true
				- ```java
				  public class Singleton {
				      private Singleton() {}
				      
				      private static class SingletonHolder {
				          private static final Singleton instance = new Singleton();
				      }
				      
				      public static Singleton getInstance() {
				          return SingletonHolder.instance;
				      }
				  }
				  ```
			- ## kotlin
			  collapsed:: true
				- ```kotlin
				  class Singleton  private constructor(){
				      companion object {
				          @JvmStatic
				          fun getInstance() = SingletonHolder.holder
				      }
				  
				      /**
				       * 静态内部类单例
				       */
				      private object SingletonHolder {
				          val holder = Singleton()
				      }
				  }
				  ```
	- ## [[#red]]==**四、枚举（1种）**==
	  collapsed:: true
		- ## 特点
			- - 这借助JDK1.5中添加的枚举来实现单例模式。
			- - 不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象。 这种方式是Effective-Java作者Josh Bloch提倡的方式
		- ## 结论:推荐使用
		- ## java
		  collapsed:: true
			- ```java
			  public enum Singleton {
			      INSTANCE;
			      
			      // 单例的其他方法和属性
			      public void doSomething() {
			          // 实现单例的操作
			      }
			  }
			  ```
		- ## kotlin
		  collapsed:: true
			- ```kotlin
			  enum class Singleton {
			      INSTANCE;
			      
			      // 单例的其他方法和属性
			      fun doSomething() {
			          // 实现单例的操作
			      }
			  }
			  ```