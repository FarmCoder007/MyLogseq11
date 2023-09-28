## 类型一、新创建的被观察者【没有上游】
	- ## 1、无延迟（比如有几秒延迟执行）无 后续（后续的其他事件 ）
		- ## SingleJust：
			- 取消订阅 是在 subscribeActual  里 自动取消Disposable.disposed(
				- ```java
				      @Override
				      protected void subscribeActual(SingleObserver<? super T> observer) {
				          observer.onSubscribe(Disposable.disposed());
				          observer.onSuccess(value);
				      }
				  ```
	- ## 2 有延迟 或者 有后续：[[Observable.interval取消原理]]
		- api:
			- ```java
			  Observable.interval(0, 1, TimeUnit.SECONDS)    // 每隔多久发一次消息
			  ```
		- ## ObservableInterval(Observable.interval返回的)被观察者
		- 原理：
			- 主要是内部创建了一个既实现Disposable，又Runnable接口的类：包装观察者IntervalObserver;
			- 它的工作原理是，通过线程池执行定时任务，run里调用自定义观察者的onNext()方法
			-
			- 取消的原理，是取消了定时任务，断开了上下游联系
- ## 类型二、  通过map操作符创建的    他们都是有上游的
	- ## 1 没有延迟没有后续的 SingleMap
		- 直接把上游给的  Disposable  传给下游了
		- 下游可以直接取消订阅
	- 2 有延迟   没有后续的  SingleDelay   他是上游交给你个消息，你延迟一下交给下游
		- 取消订阅的关键：在上游没把消息交给SingleDelay时,,直接把上游的Disposable 交给下游,  他想取消就直接取消就行
		- 在上游 把消息交给我。      我会把下游的Disposable 置换一下，当下游想取消的时候，我会在置换的自定义的Disposable  取消延时任务；
	- 3 没有延迟   有后续的 ObservableMap
		- 在发生订阅的时候，把上游给我的Disposable 包装了一层 把这个包装后传给下游了，下游取消的话 也相当于调用了 上游的Disposable.dispose()【包装就是为了做些额外的事情】
	- 4 有后续 也有延迟   ObservableDelay
		- 订阅的时候 ，我把上游交给我的Disposable包装一层 再传给下游，，包装类里的dispose() 里具体做的就是，调用上游交给我的Disposable.dispos()，同时取消我内部的延时任务