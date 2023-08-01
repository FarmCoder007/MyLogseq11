- - `onEach`操作符是什么东西，就是返回一个新的`Flow`，但是把每个上游的值都在其高级函数中执行一遍，比如这里就是println一下，然后再`emit`到新的`Flow`中，源码如下：
- ```kotlin
  public fun <T> Flow<T>.onEach(action: suspend (T) -> Unit): Flow<T> = transform { value ->
      action(value)
      return@transform emit(value)
  }
  ```
- 可以看出，这里出现了一个新的`Flow`。