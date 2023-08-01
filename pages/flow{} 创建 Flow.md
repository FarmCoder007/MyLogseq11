- 我们这里先重点看一下这里的`flow{}`高阶函数，它的作用是创建一个新的`Flow`，同时在它的lambda中我们使用`emit()`挂起函数往这个`Flow`中**发送数据**，它是一个上游操作符。
- 所以上游操作符的作用是创建一个`Flow`，然后负责往`Flow`中发送数据；类比于现实中，河流的水也是从上游产生的一样，所以上游操作符不仅要创建`Flow`，还负责发送数据。
- 我们可以简单看一下`flow{}`函数定义：
	- ```kotlin
	  public fun <T> flow(@BuilderInference block: suspend FlowCollector<T>.() -> Unit): Flow<T> 
	      = SafeFlow(block)
	  ```
- 首先它是一个顶层函数，然后`block`是函数类型，而且它的接收者是`FlowCollector`类型，根据简写约定，`block`就相当于是`FlowCollector`的成员函数，所以它可以调用`FlowCollector`类型中的方法、变量，该接口：
- ```kotlin
  public fun interface FlowCollector<in T> {
      public suspend fun emit(value: T)
  }
  ```
- 这里我们可以发现我们可以在`block`中调用`emit`也就知道原因了。
- 这里还有一个疑惑，`flow`方法不是扩展函数，但是`block`中可以调用`emit`方法，这个`emit`方法是哪个对象的呢？
- 我们简单来看一下其实现`SafeFlow`:
- ```kotlin
  private class SafeFlow<T>(private val block: suspend FlowCollector<T>.() -> Unit) : AbstractFlow<T>() {
      override suspend fun collectSafely(collector: FlowCollector<T>) {
          collector.block()
      }
  }
  ```
- `flow{}`函数会返回一个`SafeFlow`对象，然后在其实现方法`collectSafely`中会有一个`FlowCollector`类型的对象，然后可以利用该对象调用`block`，即`collector.block()`，这里也进一步说明了前面的约定观点。
- 那这个`collector`对象是什么赋值的呢？具体原理研究，等后面原理篇文章再细说。