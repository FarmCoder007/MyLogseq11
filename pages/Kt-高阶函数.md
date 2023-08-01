- ![](https://images2018.cnblogs.com/blog/1255627/201806/1255627-20180625175733663-1224851246.png)
- # 1、高阶函数介绍
  collapsed:: true
	- 在`Kotlin`中，[[#red]]==**高阶函数即指：将lamda函数用作一个函数的参数或者返回值的函数。**==
	- ## 1.1、将函数用作函数参数的情况的高阶函数
	  id:: 64c1d369-b477-42ba-af34-94be640953e9
	  collapsed:: true
		- 这里介绍字符串中的`sumBy{}`高阶函数。先看一看源码
		- ```kotlin
		  // sumBy函数的源码
		  public inline fun CharSequence.sumBy(selector: (Char) -> Int): Int {
		      var sum: Int = 0
		      for (element in this) {
		          sum += selector(element)
		      }
		      return sum
		  }
		  
		  ```
		- 源码说明：
			- 大家这里可以不必纠结`inline`，和`sumBy`函数前面的`CharSequence.`。因为这是`Koltin`中的`内联函数`与`扩展功能`。在后面的章节中会给大家讲解到的。这里主要分析高阶函数，故而这里不多做分析。
			- 该函数返回一个`Int`类型的值。并且接受了一个`selector()`函数作为该函数的参数。其中，`selector()`函数接受一个`Char`类型的参数，并且返回一个`Int`类型的值。
			- 定义一个`sum`变量，并且循环这个字符串，循环一次调用一次`selector()`函数并加上`sum`。用作累加。其中`this`关键字代表字符串本身。
		- 所以这个函数的作用是：**把字符串中的每一个字符转换为`Int`的值，用于累加，最后返回累加的值**
		- 例：
			- ```kotlin
			  val testStr = "abc"
			  val sum = testStr.sumBy { it.toInt() }
			  println(sum) //294  // 因为字符a对应的值为97,b对应98，c对应99，故而该值即为 97 + 98 + 99 = 294
			  ```
		-
	- ## 1.2、将函数用作一个函数的返回值的高阶函数。
	  collapsed:: true
		- 这里使用官网上的一个例子来讲解。`lock()`函数，先看一看他的源码实现
		  collapsed:: true
			- ```kotlin
			  fun <T> lock(lock: Lock, body: () -> T): T {
			      lock.lock()
			      try {
			          return body()
			      }
			      finally {
			          lock.unlock()
			      }
			  }
			  
			  ```
		- 源码说明：
			- 这其中用到了`kotlin`中`泛型`的知识点，这里赞不考虑。我会在后续的文章为大家讲解。
			- 从源码可以看出，该函数接受一个`Lock`类型的变量作为参数`1`，并且接受一个无参且返回类型为`T`的函数作为参数`2`.
			- 该函数的返回值为一个函数，我们可以看这一句代码`return body()`可以看出。
		- 例：使用`lock`函数，下面的代码都是伪代码，我就是按照官网的例子直接拿过来用的
			- ```kotlin
			  fun toBeSynchronized() = sharedResource.operation()
			  val result = lock(lock, ::toBeSynchronized)    
			  
			  ```
		- 其中，`::toBeSynchronized`即为对函数`toBeSynchronized()`的引用，其中关于双冒号`::`的使用在这里不做讨论与讲解。
		- 上面的写法也可以写作：
			- ```kotlin
			  val result = lock(lock, {sharedResource.operation()} )
			  
			  ```
	- ## 1-3、高阶函数的使用
		- 在上面的两个例子中，我们出现了`str.sumBy{ it.toInt }`这样的写法。其实这样的写法在前一章节`Lambda使用`中已经讲解过了。这里主要讲高阶函数中对`Lambda语法`的简写。
		- 从上面的例子我们的写法应该是这样的：
		- ```kotlin
		  str.sumBy( { it.toInt } )
		  
		  ```
		- 但是根据`Kotlin`中的约定，即当函数中只有一个函数作为参数，并且您使用了`lambda`表达式作为相应的参数，则可以省略函数的小括号`()`。故而我们可以写成：
			- ```kotlin
			  str.sumBy{ it.toInt }
			  ```
		- 还有一个约定，即当函数的最后一个参数是一个函数，并且你传递一个`lambda`表达式作为相应的参数，则可以在圆括号之外指定它。故而上面例`2`中的代码我们可写成：
			- ```kotlin
			  val result = lock(lock){
			       sharedResource.operation()
			  }
			  ```
- # 2、[[自定义高阶函数]]
- # 3、[[常用的标准高阶函数介绍]]