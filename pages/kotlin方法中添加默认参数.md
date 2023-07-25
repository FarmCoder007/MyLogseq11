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
- ## 示例2：一 函数参数默认值  代替java函数重载
	- 正常函数重载
	- ```kotlin
	      // 单参函数  里加默认值调用双参函数
	      fun toast(string: String?) {
	          toast(string, Toast.LENGTH_SHORT)
	      }
	   
	      fun toast(string: String?, duration: Int) {
	          Toast.makeText(MyApplication.currentContext, string, duration).show()
	      }
	  ```
	- JvmOverloads 注解简化
		- ```java
		      // 函数参数默认值  解决函数重载     kotlin  可直接使用  但是在java中需要使用@JvmOverloads 对java暴露重载函数
		      @JvmOverloads
		      fun toast(string: String?, duration: Int = Toast.LENGTH_SHORT) {
		          Toast.makeText(MyApplication.currentContext, string, duration).show()
		      }
		  ```