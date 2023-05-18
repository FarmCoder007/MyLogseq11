title:: kotlin1.5新特性

- # 内联类
	- 1.5.0版本后开始支持内联类，使用注解@JvmInline配合关键字value定义内联类，例子如下：
	  collapsed:: true
		- ```
		  @JvmInline
		  value class StationaryPhoneNumber(val number: String?){
		      fun addAreaCode(areaCode: String): String{
		          println("addAreaCode($areaCode)")
		          return "$areaCode-$number"
		      }
		  }
		  ```
	- 可以看到它跟数据类比较类似，这里定义一个相似的数据类来看一下区别：
	  collapsed:: true
		- ```
		  data class MobilePhoneNumber(val number: String?){
		      fun addNationCode(nationCode: String): String{
		          println("addAreaCode($nationCode)")
		          return "$nationCode-$number"
		      }
		  }
		  ```
	- 内联类字节码反编译成java代码：
		- ```
		  ```
-