title:: kt高阶-T.takeIf()函数

- # 作用
	- 传入一个你希望的一个条件，如果对象符合你的条件则返回自身，反之，则返回`null`。
- # 源码
	- ```kotlin
	  public inline fun <T> T.takeIf(predicate: (T) -> Boolean): T? {
	      contract {
	          callsInPlace(predicate, InvocationKind.EXACTLY_ONCE)
	      }
	      return if (predicate(this)) this else null
	  }
	  ```
- # 示例
	- 例： 判断一个字符串是否由某一个字符起始，若条件成立则返回自身，反之，则返回`null`
	- ```kotlin
	  val str = "kotlin"
	  
	  val result = str.takeIf {
	      it.startsWith("ko") 
	  }
	  
	  println("result = $result")
	  ```
	- 输出 result = kotlin