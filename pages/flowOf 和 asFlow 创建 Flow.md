- 除了上面方法外，还有类似于集和API的`arrayListOf`等的`flowOf`函数，以及`asFlow`把集和转换为Flow，代码如下：
- ```kotlin
  fun main() = runBlocking {
      listOf(1,2,3,4,5).asFlow()
          .filter { it > 2 }     // 中转站1
          .map { it * 2 }        // 中转站2
          .take(2)               // 中转站3
          .collect{              // 下游
              println(it)
          }
  
      flowOf(1,2,3,4,5)
          .filter { it > 2 }     // 中转站1
          .map { it * 2 }        // 中转站2
          .take(2)               // 中转站3
          .collect{              // 下游
              println(it)
          }
  }
  ```
- 这2种方法也可以创建`Flow`并且往里发送数据。
- ## asFlow
	- ```kotlin
	  public fun <T> Iterable<T>.asFlow(): Flow<T> = flow {
	      forEach { value ->
	          emit(value)
	      }
	  }
	  ```
- ## flowOf
	- ```kotlin
	  public fun <T> flowOf(vararg elements: T): Flow<T> = flow {
	      for (element in elements) {
	          emit(element)
	      }
	  }
	  ```
- 可以发现这2个方法都是调用`flow{}`高阶函数函数来实现的，并且都是遍历调用`emit`方法来完成发送数据，所以核心还是这个`emit`方法，后面文章再说其原理。