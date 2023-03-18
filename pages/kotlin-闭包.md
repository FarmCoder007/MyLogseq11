- ## 一、什么是闭包
- ## 二、闭包的使用
- ## 三、闭包和接口回调对比
  collapsed:: true
	- 1、定义接口与闭包对比
		- 接口：带返回值的函数
			- ```java
			  public interface HookListener {
			      String hook(String str);
			  }
			  ```
		- 闭包：带返回值的函数
			- ```kotlin
			  // logHook 为函数类型的闭包  输入参数为String 返回参数为String
			  logHook: (log: String) -> String?  
			  这个 作为方法入参 在 下边2定义  
			  ```
			- ```kotlin
			  // 定义一个 函数类型的变量myClosure  
			  
			  
			  val myClosure: (String) -> String = { input ->
			      // 在这里可以使用闭包中定义的变量
			      "Hello, $input!"
			  }
			  ```
	- 2、将接口类型  和 闭包类型 定义在方法入参上
		- 接口类型的方法入参：
			- ```kotlin
			      fun log(hookListener: HookListener?){
			          // 将接口存 list里
			          hookLogList.add(hookListener)
			      }
			  ```
		- 闭包类型的方法入参：
			- ```kotlin
			      fun log(logHook: (log: String) -> String?) {
			          // 将闭包 存 list里
			          hookLogList.add(logHook)
			      }
			  ```
	- 3、调用2方法的示例：写法一样
		- 设置接口：
			- ```kotlin
			         Instance().log { log: String ->
			              "$log ****** ppu=${this.getPPU()}"
			          }
			  ```
		- 设置闭包：
			- ```kotlin
			  		Instance().log { log: String ->
			              "$log ****** ppu=${this.getPPU()}"
			          }
			  ```
	- 4、获取设置的接口 和 闭包 ，进行回调的示例：
		- 接口回调：
			- ```kotlin
			  // it为 拿到的接口对象 调用 对应的方法
			  it?.hook(logBody)
			  ```
		- 闭包回调：
			- ```kotlin
			  // it为拿到的闭包，调用自身
			  it.invoke(logBody).toString()
			  ```