- Rx扩展了观察者模式用于支持数据和事件序列，添加了一些操作符，它让你可以声明式的组合这些序列，而无需关注底层的实现：如线程、同步、线程安全、并发数据结构和非阻塞IO。
  
  Observable通过使用最佳的方式访问异步数据序列填补了这个间隙
  
  |      | **单个数据**          | **多个数据**              |
  | ---- | --------------------- | ------------------------- |
  | 同步 | `T getData()`         | `Iterable<T> getData()`   |
  | 异步 | `Future<T> getData()` | `Observable<T> getData()` |
  
  Rx的Observable模型让你可以像使用集合数据一样操作异步事件流，对异步事件流使用各种简单、可组合的操作。
- #### Observable可组合
	- 对于单层的异步操作来说，Java中Future对象的处理方式是非常简单有效的，但是一旦涉及到嵌套，它们就开始变得异常繁琐和复杂。使用Future很难很好的组合带条件的异步执行流程（考虑到运行时各种潜在的问题，甚至可以说是不可能的），当然，要想实现还是可以做到的，但是非常困难，或许你可以用`Future.get()`，但这样做，异步执行的优势就完全没有了。从另一方面说，Rx的Observable一开始就是为组合异步数据流准备的。
- #### Observable更灵活
	- Rx的Observable不仅支持处理单独的标量值（就像Future可以做的），也支持数据序列，甚至是无穷的数据流。`Observable`是一个抽象概念，适用于任何场景。Observable拥有它的近亲Iterable的全部优雅与灵活。
	  
	  Observable是异步的双向push，Iterable是同步的单向pull，对比：
	  
	  | 事件     | Iterable(pull)     | Observable(push)     |
	  | -------- | ------------------ | -------------------- |
	  | 获取数据 | `T next()`         | `onNext(T)`          |
	  | 异常处理 | throws `Exception` | `onError(Exception)` |
	  | 任务完成 | `!hasNext()`       | `onCompleted()`      |
- #### Observable无偏见
  collapsed:: true
	- Rx对于对于并发性或异步性没有任何特殊的偏好，Observable可以用任何方式实现，线程池、事件循环、非阻塞IO、Actor模式，任何满足你的需求的，你擅长或偏好的方式都可以。无论你选择怎样实现它，无论底层实现是阻塞的还是非阻塞的，客户端代码将所有与Observable的交互都当做是异步的。
	  
	  **Observable是如何实现的？**
	  
	  ```java
	  public Observable<data> getData();
	  ```
	- 它能与调用者在同一线程同步执行吗？
	- 它能异步地在单独的线程执行吗？
	- 它会将工作分发到多个线程，返回数据的顺序是任意的吗？
	- 它使用Actor模式而不是线程池吗？
	- 它使用NIO和事件循环执行异步网络访问吗？
	- 它使用事件循环将工作线程从回调线程分离出来吗？
	  
	  从Observer的视角看，这些都无所谓，重要的是：使用Rx，你可以改变你的观念，你可以在完全不影响Observable程序库使用者的情况下，彻底的改变Observable的底层实现。
- #### 使用回调存在很多问题
  collapsed:: true
	- 回调在不阻塞任何事情的情况下，解决了`Future.get()`过早阻塞的问题。由于响应结果一旦就绪Callback就会被调用，它们天生就是高效率的。不过，就像使用Future一样，对于单层的异步执行来说，回调很容易使用，对于嵌套的异步组合，它们显得非常笨拙。
- #### Rx是一个多语言的实现
  collapsed:: true
	- Rx在大量的编程语言中都有实现，并尊重实现语言的风格，而且更多的实现正在飞速增加。
- #### 响应式编程
  collapsed:: true
	- Rx提供了一系列的操作符，你可以使用它们来过滤(filter)、选择(select)、变换(transform)、结合(combine)和组合(compose)多个Observable，这些操作符让执行和复合变得非常高效。
	  
	  你可以把Observable当做Iterable的推送方式的等价物，使用Iterable，消费者从生产者那拉取数据，线程阻塞直至数据准备好。使用Observable，在数据准备好时，生产者将数据推送给消费者。数据可以同步或异步的到达，这种方式更灵活。
	  
	  下面的例子展示了相似的高阶函数在Iterable和Observable上的应用
	  
	  ```java
	  // Iterable
	  getDataFromLocalMemory()
	  .skip(10)
	  .take(5)
	  .map({ s -> return s + " transformed" })
	  .forEach({ println "next => " + it })
	  
	  // Observable
	  getDataFromNetwork()
	  .skip(10)
	  .take(5)
	  .map({ s -> return s + " transformed" })
	  .subscribe({ println "onNext => " + it })
	  ```
	  
	  Observable类型给GOF的观察者模式添加了两种缺少的语义，这样就和Iterable类型中可用的操作一致了：
	  
	  1. 生产者可以发信号给消费者，通知它没有更多数据可用了（对于Iterable，一个for循环正常完成表示没有数据了；对于Observable，就是调用观察者的`onCompleted`方法）
	  2. 生产者可以发信号给消费者，通知它遇到了一个错误（对于Iterable，迭代过程中发生错误会抛出异常；对于Observable，就是调用观察者(Observer)的`onError`方法）
	  
	  有了这两种功能，Rx就能使Observable与Iterable保持一致了，唯一的不同是数据流的方向。任何对Iterable的操作，你都可以对Observable使用。