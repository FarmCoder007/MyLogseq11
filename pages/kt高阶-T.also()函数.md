title:: kt高阶-T.also()函数

- # 源码
	- ```kotlin
	  public inline fun <T> T.also(block: (T) -> Unit): T {
	      contract {
	          callsInPlace(block, InvocationKind.EXACTLY_ONCE)
	      }
	      block(this)
	      return this
	  }
	  ```
	- 从上面的源码在结合`T.apply`函数的源码我们可以看出：
	- **[[#red]]==T.also函数中的参数block函数传入了自身对象。故而这个函数的作用是用用block函数调用自身对象，最后在返回自身对象==**
- 这里举例一个简单的例子，并用实例说明其和`T.apply`的区别
- ## 示例
  collapsed:: true
	- ```kotlin
	  "kotlin".also {
	      println("结果：${it.plus("-java")}")
	  }.also {
	      println("结果：${it.plus("-php")}")
	  }
	  
	  "kotlin".apply {
	      println("结果：${this.plus("-java")}")
	  }.apply {
	      println("结果：${this.plus("-php")}")
	  }
	  ```
	- 他们的输出结果是相同的：
	- ```kotlin
	  结果：kotlin-java
	  结果：kotlin-php
	  
	  结果：kotlin-java
	  结果：kotlin-php
	  ```
	- 从上面的实例我们可以看出，他们的区别在于，`T.also`中只能使用`it`调用自身,而`T.apply`中只能使用`this`调用自身。因为在源码中`T.also`是执行`block(this)`后在返回自身。而`T.apply`是执行`block()`后在返回自身。这就是为什么在一些函数中可以使用`it`,而一些函数中只能使用`this`的关键所在