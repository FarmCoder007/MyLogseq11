title:: kt高阶-T.takeUnless函数

- # 作用
	- 这个函数的作用和`T.takeIf()`函数的作用是一样的。只是和其的逻辑是相反的
	- 传入一个你希望的一个条件，如果对象符合你的条件则返回`null`，反之，则返回自身
- # 源码
	- ```kotlin
	  public inline fun <T> T.takeUnless(predicate: (T) -> Boolean): T? {
	      contract {
	          callsInPlace(predicate, InvocationKind.EXACTLY_ONCE)
	      }
	      return if (!predicate(this)) this else null
	  }
	  ```
- 这里就举和`T.takeIf()`函数中一样的例子，看他的结果和`T.takeIf()`中的结果是不是相反的。
- 例：
	- ```kotlin
	  val str = "kotlin"
	  
	  val result = str.takeUnless {
	      it.startsWith("ko") 
	  }
	  
	  println("result = $result")  // result = null
	  ```
	-