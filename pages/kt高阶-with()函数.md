- ### 其实`with()`函数和`T.run()`函数的作用是相同的，我们这里看下其实现源码：
- ```kotlin
  public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
      contract {
          callsInPlace(block, InvocationKind.EXACTLY_ONCE)
      }
      return receiver.block()
  }
  
  ```
- 这里我们可以看出和`T.run()`函数的源代码实现没有太大的差别。故而这两个函数的区别在于：
	- >1 ` with`是正常的高阶函数，`T.run()`是扩展的高阶函数。
	- > 2 `with`函数的返回值指定了`receiver`为接收者。
- ## 示例1
	- 1、故而上面的`T.run()`函数的列子我也可用`with`来实现相同的效果：
		- ```kotlin
		  val str = "kotlin"
		  with(str) {
		      println( "length = ${this.length}" )
		      println( "first = ${first()}")
		      println( "last = ${last()}" )
		  }
		  ```
	- 输出结果为：
		- ```kotlin
		  length = 6
		  first = k
		  last = n
		  ```
- ## 示例2、为`TextView`设置属性，也可以用它来实现。这里我就不举例了。
	- 在上面举例的时候，都是正常的列子，这里举一个特例：当我的对象可为`null`的时候，看两个函数之间的便利性。
	- 例：
		- ```kotlin
		  val newStr : String? = "kotlin"
		  
		  with(newStr){
		      println( "length = ${this?.length}" )
		      println( "first = ${this?.first()}")
		      println( "last = ${this?.last()}" )
		  }
		  
		  newStr?.run {
		      println( "length = $length" )
		      println( "first = ${first()}")
		      println( "last = ${last()}" )
		  }
		  ```