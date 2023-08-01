- ## 一、语法
  collapsed:: true
	- ## 1、无参数的情况
		- >    val/var 变量名 = { 操作的代码 }
		- ```java
		      // 源代码
		     fun test(){ println("无参数") }
		    
		      // lambda代码
		      val test = { println("无参数") }
		  
		      // 调用
		      test()  => 结果为：无参数
		  
		  ```
	- ## 2、有参数的情况,这里举例一个两个参数的例子，
		- ### 2-1：写法1
		- >val/var 变量名 : (参数的类型，参数类型，...) -> 返回值类型 = {参数1，参数2，... -> 操作参数的代码 }
		- ### 2-2、写法2，2-1可等价于
		- > // 此种写法：即表达式的返回值类型会根据操作的代码自推导出来。
		    val/var 变量名 = { 参数1 ： 类型，参数2 : 类型, ... -> 操作参数的代码 }
		- ```kotlin
		      // 源代码
		      fun test(a : Int , b : Int) : Int{
		          return a + b
		      }
		  
		      // lambda
		      val test : (Int , Int) -> Int = {a , b -> a + b}
		      // 或者
		      val test = {a : Int , b : Int -> a + b}
		      
		      // 调用
		      test(3,5) => 结果为：8
		  
		  ```
	- ## 3、lambda表达式作为函数中的参数的时候
		- >  fun test(a : Int, 参数名 : (参数1 ： 类型，参数2 : 类型, ... ) -> 表达式返回类型){
		          ...
		      }
		- ```kotlin
		      // 源代码
		      fun test(a : Int , b : Int) : Int{
		          return a + b
		      }
		  
		      fun sum(num1 : Int , num2 : Int) : Int{
		          return num1 + num2
		      }
		  
		      // 调用
		      test(10,sum(3,5)) // 结果为：18
		  
		      // lambda
		      fun test(a : Int , b : (num1 : Int , num2 : Int) -> Int) : Int{
		          return a + b.invoke(3,5)
		      }
		  
		      // 调用
		      test(10,{ num1: Int, num2: Int ->  num1 + num2 })  // 结果为：18
		  
		  ```
- ## 二、特点
  collapsed:: true
	- >`lambda`表达式总是被大括号括着。
	- >定义完整的`Lambda`表达式如上面实例中的语法2，它有其完整的参数类型标注，与表达式返回值。当我们把一些类型标注省略的情况下，就如上面实例中的语法2的另外一种类型。当它推断出的返回值类型不为'Unit'时，它的返回值即为`->`符号后面代码的最后一个（或只有一个）表达式的类型。
	- >在上面例子中语法3的情况表示为：`高阶函数`，当`Lambda`表达式作为其一个参数时，只为其表达式提供了参数类型与返回类型，所以，我们在调用此高阶函数的时候我们要为该`Lambda`表达式写出它的具体实现。
	- >`invoke()`函数：表示为通过`函数变量`调用自身，因为上面例子中的变量`b`是一个匿名函数。
- ## 三、实战
  collapsed:: true
	- ## 3-1、[[lambda -it]]
	- ## 3-2、 [[lambda下划线（_）]]
	- ## 3-3、[[匿名函数]]
	- ## 3-4、[[带接收者的函数字面值]]
	- ## 3-5[[kotlin-闭包]]
	- ## lamda函数示例
	  collapsed:: true
		- ```kotlin
		      var m06 : (Int, Int) -> Int = {number1, number2 -> number1 + number2}
		      println("m06:${m06(9, 9)}")
		  
		      var m07 = { number1: Int , number2: Int -> number1.toDouble() + number2.toDouble()}
		      println("m07:${m07(100, 100)}")
		  
		      var m08 : (String, String) -> Unit = {aString, bString -> println("a:$aString,  b:$bString")}
		      m08("李元霸", "王五")
		  
		      var m09 : (String) -> String = {str -> str}
		      println("m09:${m09("降龙十八掌")}")
		  
		      // 单参数可以用 it
		      var m10 : (Int) -> Unit = {
		          when(it) {
		              1 -> println("你是一")
		              in 20..30 -> println("你是 二十 到 三十")
		              else -> println("其他的数字")
		          }
		      }
		      m10(29)
		  
		      // 多参数不能用it 
		      var m11 : (Int, Int, Int) -> Unit = { n1, n2, n3 ->
		          println("n1:$n1, n2:$n2, n3:$n3")
		      }
		      m11(29,22,33)
		  
		      // 闭包
		      var m12 = { println("我就是m12函数，我就是我") }
		      m12()
		      // 带参数闭包
		      val m13 = {sex: Char -> if (sex == 'M') "代表是男的" else "代表是女的"}
		      println("m13:${m13('M')}")
		  
		      // 覆盖操作
		      var m14 = {number: Int -> println("我就是m14  我的值: $number")}
		      m14 = {println("覆盖  我的值: $it")}
		      m14(99)
		  
		      // 需求：我想打印， 并且，我还想返回值，就是 我什么都想要
		      var m15 = { number: Int -> println("我想打印: $number")
		          number + 1000000
		      }
		      println("m15:${m15(88)}")
		  ```
- ## 示例
	- 示例  ：循环遍历 使用lambad
	  collapsed:: true
		- ```kotlin
		  var strList = mutableListOf("22", "33", "55")
		   // 正常遍历
		      for (str in strList) {
		          println(str)
		      }
		      // 使用lambda
		      // lambda  1 声明在{}里
		      //         2 des: String -> println(des)      格式 参数名：参数类型 ->  函数体
		      strList.forEach({
		          des: String -> println(des)
		      })
		      // 注1： 在kotlin中  如果函数 最后一个参数是lambda  需要把lambda移到最外边
		      strList.forEach(){
		          des: String -> println(des)
		      }
		      // 注2： 在kotlin中  如果函数传入参数 只有一个lambda的话  （）可以省略
		      strList.forEach{
		          des: String -> println(des)
		      }
		      // 由于kotlin 有类型推断  遍历的是string集合  类型可以省略
		      strList.forEach{
		          des -> println(des)
		      }
		      // 注3 ： lambda只有一个传入参数的话  参数可以省略  用隐式的it来访问这个参数
		      strList.forEach{
		         println(it)
		      }
		  ```
- ## 教程
	- [lambda 表达式](https://www.cnblogs.com/Jetictors/p/8647888.html)