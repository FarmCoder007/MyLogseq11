title:: kt高阶-T.apply()函数

- # 作用：
	- T,apply执行完了block()函数后，[[#red]]==**返回了自身对象**==。而T.[[#red]]==**run是返回了执行的结果**==。
- # 源码
	- 我们先看下`T.apply()`函数的源码：
	- ```kotlin
	  public inline fun <T> T.apply(block: T.() -> Unit): T {
	      contract {
	          callsInPlace(block, InvocationKind.EXACTLY_ONCE)
	      }
	      block()
	      return this
	  }
	  ```
	- 从`T.apply()`源码中在结合前面提到的`T.run()`函数的源码我们可以得出,这两个函数的逻辑差不多，唯一的区别是[[#red]]==**`T,apply`执行完了`block()`函数后**==，[[#red]]==**返回了自身对象**==。而`T.run`是返回了执行的结果。
	- 故而： `T.apply`的作用除了实现能实现`T.run`函数的作用外，还可以后续的再对此操作。下面我们看一个例子：
	-
	-
- # 示例1
	- 例：为`TextView`设置属性后，再设置点击事件等
	- ```kotlin
	  val mTvBtn = findViewById<TextView>(R.id.text)
	  mTvBtn.apply{
	      text = "kotlin"
	      textSize = 13f
	      ...
	  }.apply{
	      // 这里可以继续去设置属性或一些TextView的其他一些操作
	  }.apply{
	      setOnClickListener{ .... }
	  }
	  ```
- # 示例2：或者：设置为`Fragment`设置数据传递
	- ```kotlin
	  // 原始方法
	  fun newInstance(id : Int , name : String , age : Int) : MimeFragment{
	          val fragment = MimeFragment()
	          fragment.arguments.putInt("id",id)
	          fragment.arguments.putString("name",name)
	          fragment.arguments.putInt("age",age)
	          
	          return fragment
	  }
	  
	  // 改进方法
	  fun newInstance(id : Int , name : String , age : Int) = MimeFragment().apply {
	          arguments.putInt("id",id)
	          arguments.putString("name",name)
	          arguments.putInt("age",age)
	  }
	  ```