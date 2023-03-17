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
			- ```kotlin
			  ```
			- ```kotlin
			  // 定义一个 函数类型的变量myClosure  
			  
			  
			  val myClosure: (String) -> String = { input ->
			      // 在这里可以使用闭包中定义的变量
			      "Hello, $input!"
			  }
			  ```
	- 接口回调：
	-