- # 1、出现场景
	- - `it`并不是`Kotlin`中的一个关键字(保留字)。
	- - it是在当一个高阶函数中Lambda表达式的[[#red]]==**参数只有一个的时候可以使用it来使用此参数**==。it可表示为单个参数的隐式名称，是Kotlin语言约定的。
- # 2、示例
	- ## 例1
		- ```kotlin
		  val it : Int = 0  // 即it不是`Kotlin`中的关键字。可用于变量名称
		  ```
	- ## 例2：单个参数的隐式名称
		- ```kotlin
		  // 这里举例一个语言自带的一个高阶函数filter,此函数的作用是过滤掉不满足条件的值。
		  val arr = arrayOf(1,3,5,7,9)
		  // 过滤掉数组中元素小于2的元素，取其第一个打印。这里的it就表示每一个元素。
		  println(arr.filter { it < 5 }.component1())   
		  
		  ```
	- ## 例3：
		- ```kotlin
		   fun test(num1 : Int, bool : (Int) -> Boolean) : Int{
		     return if (bool(num1)){ num1 } else 0
		  }
		  
		  println(test(10,{it > 5}))
		  println(test(4,{it > 5}))
		  
		  ```
		- 输出结果为：
		- ```kotlin
		  10
		  0
		  
		  ```