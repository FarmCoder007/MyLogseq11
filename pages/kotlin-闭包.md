- ## 一、什么是闭包
- ## 二、闭包的使用
- ## 三、闭包和接口回调对比
	- 1、定义接口与闭包对比
		- 接口：带返回值的函数
			- ```java
			  public interface HookListener {
			      String hook(String str);
			  }
			  ```
		- 闭包：带返回值的函数
		  collapsed:: true
			- ```kotlin
			  // logHook 为函数类型的闭包  输入参数为String 返回参数为String
			  logHook: (log: String) -> String?
			  ```
			- ```kotlin
			  // 定义一个 函数类型的变量myClosure  
			  
			  
			  val myClosure: (String) -> String = { input ->
			      // 在这里可以使用闭包中定义的变量
			      "Hello, $input!"
			  }
			  ```
	- 2、将接口类型  和 闭包类型 定义在方法入参上
		-
	-