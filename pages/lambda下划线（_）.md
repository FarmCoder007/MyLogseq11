- > 在使用`Lambda`表达式的时候，可以用下划线(`_`)表示未使用的参数，表示不处理这个参数。
- ## 使用场景
	- 同时在遍历一个`Map`集合的时候，这当非常有用
- ## 示例
	- ```kotlin
	  val map = mapOf("key1" to "value1","key2" to "value2","key3" to "value3")
	  
	  map.forEach{
	       key , value -> println("$key \t $value")
	  }
	  
	  // 不需要key的时候
	  map.forEach{
	      _ , value -> println("$value")
	  }
	  
	  ```
	- 输出
		- ```kotlin
		  key1 	 value1
		  key2 	 value2
		  key3 	 value3
		  value1
		  value2
		  value3
		  ```