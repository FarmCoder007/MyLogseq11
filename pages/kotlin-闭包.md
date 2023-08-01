- ## 一、什么是闭包
	- 所谓`闭包`，即是[[#red]]==**函数中包含函数**==，这里的函数我们可以包含(`Lambda`表达式，匿名函数，局部函数，对象表达式)。我们熟知，函数式编程是现在和未来良好的一种编程趋势。故而`Kotlin`也有这一个特性。
	- 我们熟知，`Java`是不支持闭包的，`Java`是一种面向对象的编程语言，在`Java`中，`对象`是他的一等公民。`函数`和`变量`是二等公民。
	- `Kotlin`中支持闭包，`函数`和`变量`是它的一等公民，而`对象`则是它的二等公民了。
- ## 二、示例
  collapsed:: true
	- 实例：看一段`Java`代码
		- ```kotlin
		  public class TestJava{
		  
		      private void test(){
		          private void test(){        // 错误，因为Java中不支持函数包含函数
		  
		          }
		      }
		  
		      private void test1(){}          // 正确，Java中的函数只能包含在对象中+
		  }
		  
		  ```
	- kotlin
		- ```kotlin
		  fun test1(){
		      fun test2(){   // 正确，因为Kotlin中可以函数嵌套函数
		          
		      }
		  }
		  ```
- ## 四、下面我们讲解`Kotlin`中几种闭包的表现形式。
  collapsed:: true
	- ## 4-1、携带状态
		- 例：让函数返回一个函数，并携带状态值
		- ```kotlin
		  fun test(b : Int): () -> Int{
		      var a = 3
		      return fun() : Int{
		          a++
		          return a + b
		      }
		  }
		  
		  val t = test(3)
		  println(t())
		  println(t())
		  println(t())
		  7
		  8
		  9
		  ```
	- ## 4-2、引用外部变量，并改变外部变量的值
		- ```kotlin
		  var sum : Int = 0
		  val arr = arrayOf(1,3,5,7,9)
		  arr.filter { it < 7  }.forEach { sum += it }
		  
		  println(sum) //9
		  
		  
		  ```
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
-