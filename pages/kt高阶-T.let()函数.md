title:: kt高阶-T.let()函数

- # 作用
	- 在前面讲解`空安全、可空属性`章节中，我们讲解到可以使用`T.let()`函数来规避空指针的问题
- # 源码
	- ```kotlin
	  public inline fun <T, R> T.let(block: (T) -> R): R {
	      contract {
	          callsInPlace(block, InvocationKind.EXACTLY_ONCE)
	      }
	      return block(this)
	  }
	  ```
	- 从上面的源码中我们可以得出，它其实和`T.also`以及`T.apply`都很相似。而`T.let`的作用也不仅仅在使用`空安全`这一个点上。用`T.let`也可实现其他操作
- # 示例
	- ```kotlin
	  "kotlin".let {
	      println("原字符串：$it")         // kotlin
	      it.reversed()
	  }.let {
	      println("反转字符串后的值：$it")     // niltok
	      it.plus("-java")
	  }.let {
	      println("新的字符串：$it")          // niltok-java
	  }
	  
	  "kotlin".also {
	      println("原字符串：$it")     // kotlin
	      it.reversed()
	  }.also {
	      println("反转字符串后的值：$it")     // kotlin
	      it.plus("-java")
	  }.also {
	      println("新的字符串：$it")        // kotlin
	  }
	  
	  "kotlin".apply {
	      println("原字符串：$this")     // kotlin
	      this.reversed()
	  }.apply {
	      println("反转字符串后的值：$this")     // kotlin
	      this.plus("-java")
	  }.apply {
	      println("新的字符串：$this")        // kotlin
	  }
	  
	  ```
	- 输出结果看是否和注释的结果一样呢：
		- ```kotlin
		  原字符串：kotlin
		  反转字符串后的值：niltok
		  新的字符串：niltok-java
		  
		  原字符串：kotlin
		  反转字符串后的值：kotlin
		  新的字符串：kotlin
		  
		  原字符串：kotlin
		  反转字符串后的值：kotlin
		  新的字符串：kotlin
		  ```