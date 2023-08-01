title:: 高阶函数T.run和with区别

- ## 功能相似：都可以引用当前对象的上下文用this
- ## 1、` with`是正常声明的高阶函数，`T.run()`是为所有类型扩展的高阶函数。
- ## 2、操作可空对象时，T.run()可以先空判断
	- ```kotlin
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