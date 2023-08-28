## 1、Rxjava 框架结构
collapsed:: true
	- RxJava 的整体结构是⼀条链，其中：
	  1. 链的最上游：⽣产者 Observable
	  2. 链的最下游：观察者 Observer
	  3. 链的中间：各个中介节点，既是下游的 Observable ，⼜是上游的 Observer
- ## 2、[[标准观察者和rxJava观察者的区别？]]
- ## 2、Observer 观察者接口  回调方法时机和作用，，以及 其他方法的概念
  collapsed:: true
	- ```kotlin
	          // 创建single 对象   他其实是个被观察者
	          var single = Single.just("1")
	          // single（被观察者）订阅 一个单一观察者SingleObserver
	          single.subscribe(object:SingleObserver<String?>{
	              // 订阅开始之前在主线程被回调：可以用来初始化  
	              override fun onSubscribe(d: Disposable?) {
	   
	              }
	   
	              /**
	               *  成功的时候
	               */
	              override fun onSuccess(t: String?) {
	                  textView.text = t
	              }
	              // 失败的时候回调    
	              override fun onError(e: Throwable?) {
	   
	              }
	   
	          })
	  ```
	- # onSubscribe
	  collapsed:: true
		- 订阅开始之前在主线程被回调：可以用来初始化操作
		- 里的参数Disposable（丢弃的意思）：  取消订阅 Disposable.dispose()
			- 作用1 ：就是可以用这个断开上下游联系，取消订阅，  【实际上是从下游观察者（SingleObserver）角度，丢弃你上游的被观察者。。出发点是由下游发起的 】
			- 作用2： 同时让上游停止生产的工作
	- # onSuccess：成功时候回调
	- # onError：失败的时候回调
	- # subscribeOn(Schedulers.io())
	  collapsed:: true
		- 作用：订阅到io线程 ，这样上游生产者【被观察者】才能在后台线程做耗时任务
	- # observeOn(AndroidSchedulers.mainThread())
		- 作用： 观察切回前台线程，将生产者在io线程的数据      切到主线程回调
- ## 3、rxjava订阅机制是怎样的？
	- ## 下面以Single.just为例来说明一个最简单的完整的订阅机制：
	- 1、首先最简单的订阅机制包含3个概念
		- 上游被观察者：
		- 下游观察者
		- 发起订阅
	- 2、Single这里通过Single.just(1)创建一个上层的被观察对象[[#red]]==**SingleJust**==
	- 3、SingleJust调用subscribe 发起订阅时，传入一个自定义下游观察者对象，SingleObserver
	- 4、subscribe 内部最终会调用SingleJust的subscribeActual方法完成订阅与数据传递
		- ```java
		      @Override
		      protected void subscribeActual(SingleObserver<? super T> observer) {
		          observer.onSubscribe(Disposable.disposed());
		          observer.onSuccess(value);
		      }
		  ```
	- 整体流程从订阅函数开始向上传递订阅，一直到顶层被观察者
	- 发送数据是向下从被观察者，到最下边的观察者。整体U型
- ## 4、map操作符原理
  collapsed:: true
	- 代码
	  collapsed:: true
		- ```java
		          // 创建single对象    这里存入的是int
		          var single: Single<Int> = Single.just(1)
		          // 所以需要操作符  map将int 转成 String
		          val singleString = single.map(object : Function<Int, String> {
		              override fun apply(t: Int?): String {
		                  return t.toString()
		              }
		          })
		          
		          // 订阅  接收的是String
		          singleString.subscribe(object : SingleObserver<String?> {
		              override fun onSubscribe(d: Disposable?) {}
		              override fun onSuccess(t: String?) { textView.text = t}
		              override fun onError(e: Throwable?) { }
		          })
		  ```
	- ## [[Rxjava3-map操作符原理]]
- ## 5、[[Rxjava3取消订阅的原理]]
  collapsed:: true
	- 调用 **Disposable.**dispose()取消订阅
		- 就是让上游被观察者，停止发送数据
		- 或者有中间层【通过map创建的被观察者】的情况 让中间层的定时任务取消
		- 以及断开上下游的联系
- ## 6、线程切换原
  collapsed:: true
	- ### [[subscribeOn (Schedulers.io())怎么切线程？]]
	- ### [[observeOn(AndroidSchedulers.mainThread())]]
- ## 7、[[背压是什么？]]
- ## 8、[[背压如何解决？]]