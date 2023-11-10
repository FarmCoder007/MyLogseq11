## 一、简介
- 使用inline关键字声明的函数是「内联函数」，在「编译时」会将「内联函数」中的函数体直接插入到调用处。所以在写内联函数的时候需要注意，尽量将内联函数中的代码行数减少！
- ## 二、作用 ：
	- 一小步优点：    减少一层函数调用栈     【意义不大】
	- 最大优点 ： 在接收类型是函数类型的方法 时
		- 不声明成内联函数  每次调用都会生成一个对象 ，
		- 而对于传入参数是函数类型的函数 最大的优点是减少对象的产生
	- 注意：  尽量将内联函数中的代码行数减少
- ## 三、使用场景：
	- 内联函数  适合 传入参数类型  是函数类型的参数
- ## 四、reified关键字
	- ### 作用
		- `reified` 主要用于内联函数（inline functions）中，以在运行时保留泛型类型信息。不加的话，正常泛型信息运行时会被擦除。
	- ### 举例：
	  collapsed:: true
		- 典型的例子是在使用泛型函数中获取泛型类型的实际类。例如，以下是一个使用 `reified` 的例子：
		- ```kotlin
		  inline fun <reified T> exampleFunction(value: T) {
		      // 这里运行时可以拿到泛型的具体类型，
		      val type = T::class.java
		      println("The actual type is: $type")
		  }
		  
		  fun main() {
		      exampleFunction(42)
		      exampleFunction("Hello")
		  }
		  
		  ```
		- `reified T` 允许在 `exampleFunction` 内部使用 `T::class.java` 来获取泛型参数 `T` 的实际类型，而不会在运行时丢失类型信息。
	-