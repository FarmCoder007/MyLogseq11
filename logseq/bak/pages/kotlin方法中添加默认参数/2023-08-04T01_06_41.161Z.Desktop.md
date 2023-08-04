- ## 一、在类的方法中应用@JvmOverloads
	- kotlin调用自身的中方法，默认参数是可以不传递的,相当于方法的重载，如何让Java调用kotlin中默认参数方法，可以使用
	- ```kotlin
	  @JvmOverloads
	  fun test(name : String = "kd") {
	    
	  }
	  //Java调用
	  Test.INSTANCE.test();
	  ```
	-
- ## 二、