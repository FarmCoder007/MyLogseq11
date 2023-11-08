# 概念题
collapsed:: true
	- ## 1、Rxjava 框架结构
		- RxJava 的整体结构是⼀条链，其中：
		  1. 链的最上游：⽣产者 Observable
		  2. 链的最下游：观察者 Observer
		  3. 链的中间：各个中介节点，既是下游的 Observable ，⼜是上游的 Observer
	- ## 2、[[标准观察者和rxJava观察者的区别？]]
		- 标准观察者
			- 一个被观察者，多个观察者，并且需要被观察者发送改变通知后，所有的观察者才能察觉到
		- rxjava发布订阅模式
			- 多个被观察者，一个观察者。并且需要起点和终点在==**订阅一次后**==，才发出改变通知，终点观察者才能观察到
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
			- 订阅开始之前在主线程被回调：可以用来初始化操作
			- 里的参数Disposable（丢弃的意思）：  取消订阅 Disposable.dispose()
				- 作用1 ：就是可以用这个断开上下游联系，取消订阅，  【实际上是从下游观察者（SingleObserver）角度，丢弃你上游的被观察者。。出发点是由下游发起的 】
				- 作用2： 同时让上游停止生产的工作
		- # onSuccess：成功时候回调
		- # onError：失败的时候回调
		- # subscribeOn(Schedulers.io())
			- 作用：订阅到io线程 ，这样上游生产者【被观察者】才能在后台线程做耗时任务
		- # observeOn(AndroidSchedulers.mainThread())
			- 作用： 观察切回前台线程，将生产者在io线程的数据      切到主线程回调
	- ## 7、[[背压是什么？]]
		- [[#red]]==**Backpressure 其实是一种现象：**==在数据流从上游生产者向下游消费者传输的过程中，[[#red]]==**上游生产速度大于下游消费速度，导致下游的 Buffer 溢出**==，这种现象就叫做 Backpressure 出现
	- ## 8、[[背压如何解决？]]
		- 1、控制发送者的速度
		- 2、缓存事件
		- 3、丢弃事件
		- rxjava提供了些背压操作符：
			- 的基本原理都是[[#red]]==**采用缓存或丢弃策略，**==来[[#red]]==**调节被观察者发射数据的频率**==
- # 原理题
	- ## 1、rxjava订阅机制是怎样的？
		- ### 下面以Single.just为例来说明一个最简单的完整的订阅机制：
			- ```java
			          Single.just("1").subscribe(new SingleObserver<String>() {
			              @Override
			              public void onSubscribe(@NonNull Disposable d) {
			                  
			              }
			  
			              @Override
			              public void onSuccess(@NonNull String s) {
			  
			              }
			  
			              @Override
			              public void onError(@NonNull Throwable e) {
			  
			              }
			          });
			  ```
			- 1、Single这里通过Single.just(1)创建一个上层的被观察对象[[#red]]==**SingleJust**==
			- 2、SingleJust调用subscribe 发起订阅时，传入一个自定义下游观察者对象，SingleObserver
			- 3、subscribe 内部最终会调用SingleJust的subscribeActual方法完成订阅与数据传递
				- ```java
				      @Override
				      protected void subscribeActual(SingleObserver<? super T> observer) {
				          observer.onSubscribe(Disposable.disposed());
				          observer.onSuccess(value);
				      }
				  ```
			- 4、SingleObserver相应的方法进行接收。
		- 整体流程从订阅函数开始向上传递订阅，一直到顶层被观察者
		- 发送数据是向下从被观察者，到最下边的观察者。 {{cloze 整体U型}}
		  id:: 64cc4f24-4b36-4613-936e-bbcf586967a9
	- ## 2、map操作符原理
		- 代码
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
	- ## 3、[[Rxjava3取消订阅的原理]]
		- 调用 **Disposable.**dispose()取消订阅
			- 就是让上游被观察者，停止发送数据
			- 或者有中间层【通过map创建的被观察者】的情况 让中间层的定时任务取消
			- 以及断开上下游的联系
	- ## 4、线程切换原理
		- ### [[subscribeOn (Schedulers.io())怎么切线程？]]
		- ### [[observeOn(AndroidSchedulers.mainThread())]]