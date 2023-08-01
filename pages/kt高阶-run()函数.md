- `run`函数这里分为两种情况讲解，因为在源码中也分为两个函数来实现的。采用不同的`run`函数会有不同的效果。
- # 1、run()
  collapsed:: true
	- ## 我们看下其源码：
		- ```kotlin
		  public inline fun <R> run(block: () -> R): R {
		      contract {
		          callsInPlace(block, InvocationKind.EXACTLY_ONCE)
		      }
		      return block()
		  }
		  ```
		- 关于`contract`这部分代码小生也不是很懂其意思。在一些大牛的`blog`上说是其编辑器对上下文的推断。但是我也不知道对不对，因为在官网中，对这个东西也没有讲解到。不过这个单词的意思是`契约，合同`等等意思。我想应该和这个有关。在这里我就不做深究了。主要讲讲`run{}`函数的用法其含义。
		- 这里我们只关心`return block()`这行代码。从源码中我们可以看出，`run`函数仅仅是执行了我们的`block()`，即一个`Lambda`表达式，而后返回了执行的结果。
	- ## 用法1：
	  collapsed:: true
		- > 当我们需要执行一个`代码块`的时候就可以用到这个函数,并且这个代码块是独立的。即我可以在`run()`函数中写一些和项目无关的代码，因为它不会影响项目的正常运行。
		- 例: 在一个函数中使用
			- ```kotlin
			  private fun testRun1() {
			      val str = "kotlin"
			  
			      run{
			          val str = "java"   // 和上面的变量不会冲突
			          println("str = $str")
			      }
			  
			      println("str = $str")
			  }    
			  
			  str = java
			  str = kotlin
			  
			  ```
	- ## **用法2：**
	  collapsed:: true
		- >因为`run`函数执行了我传进去的`lambda`表达式并返回了执行的结果，所以当一个业务逻辑都需要执行同一段代码而根据不同的条件去判断得到不同结果的时候。可以用到`run`函数
		- 都要获取字符串的长度。
			- ```kotlin
			  val index = 3
			  val num = run {
			      when(index){
			          0 -> "kotlin"
			          1 -> "java"
			          2 -> "php"
			          3 -> "javaScript"
			          else -> "none"
			      }
			  }.length
			  println("num = $num") // num = 10
			  
			  
			  ```
- # 2、T.run()
	- 其实`T.run()`函数和`run()`函数差不多，关于这两者之间的差别我们看看其源码实现就明白了：
	- ```kotlin
	  public inline fun <T, R> T.run(block: T.() -> R): R {
	      contract {
	          callsInPlace(block, InvocationKind.EXACTLY_ONCE)
	      }
	      return block()
	  }
	  ```
	- 从源码中我们可以看出，`block()`这个函数参数是一个扩展在`T`类型下的函数。这说明我的`block()`函数可以可以使用当前对象的上下文。所以**当我们传入的`lambda`表达式想要使用当前对象的上下文的时候，我们可以使用这个函数。**
	- ## **用法：**
		- ```kotlin
		  val str = "kotlin"
		  str.run {
		      println( "length = ${this.length}" )
		      println( "first = ${first()}")
		      println( "last = ${last()}" )
		  }
		  输出
		  length = 6
		  first = k
		  last = n
		  ```
		- 在其中，可以使用`this`关键字，因为在这里它就代码`str`这个对象，也可以省略。因为在源码中我们就可以看出，`block`()
		  就是一个`T`类型的扩展函数。
	- ## 这在实际的开发当中我们可以这样用：
		- 例： 为`TextView`设置属性。
		- ```kotlin
		  val mTvBtn = findViewById<TextView>(R.id.text)
		  mTvBtn.run{
		      text = "kotlin"
		      textSize = 13f
		      ...
		  }
		  ```