- ViewModel的RxJava扩展
- 在ViewModel中我们可以使用RxJava进行网络请求，接口如下：
  collapsed:: true
	- ```
	  interface RxJavaHttpServiceDemo {
	      fun httpRx1(infoId: String): rx.Observable<API<String>>
	  
	      fun httpRx2(infoId: String): io.reactivex.Observable<API<String>>
	  }
	  ```
- 当在ViewModel中进行网络请求时，为了能够在ViewModel被销毁时取消请求，需要保存对应的subscription到CompositeSubscription中，然后在ViewModel.onCleared()方法中取消请求，具体样例代码如下：
  collapsed:: true
	- ```
	  class RxJava1Demo : ViewModel() {
	  
	      private val subscriptions = CompositeSubscription();
	  
	      override fun onCleared() {
	          super.onCleared()
	          subscriptions.clear()
	      }
	  
	      fun legacy() {
	          val subscription = WbuNetEngine.ins().get(RxJavaHttpServiceDemo::class.java)
	              .httpRx1("")
	              .subscribe(object : SubscriberAdapter<API<String>>() {
	  
	                  override fun onNext(t: API<String>?) {
	                      super.onNext(t)
	                  }
	              })
	          subscriptions.add(subscription)
	      }
	  
	  }
	  ```
- 为了简化代码编写、避免遗漏取消网络请求，针对RxJava开发了rx1/rx2模板函数，可以在ViewModel销毁时统一取消处理RxJava操作，代码如下所示：
- 注意： 针对RxJava 1.x版本使用rx1函数，针对RxJava 2.x版本使用rx2函数。
  collapsed:: true
	- ```
	  class RxJava1Demo : ViewModel() {
	  
	      fun autoCancelRx1() {
	          rx1 {
	              WbuNetEngine.ins().get(RxJavaHttpServiceDemo::class.java)
	                  .httpRx1("")
	                  .subscribe(object : SubscriberAdapter<API<String>>() {
	  
	                      override fun onNext(t: API<String>?) {
	                          super.onNext(t)
	                      }
	                  })
	          }
	      }
	  
	      fun autoCancelRx2() {
	          rx2 {
	              val disposable = object : ResourceObserver<API<String>>() {
	  
	                  override fun onNext(t: API<String>) {
	                      TODO("Not yet implemented")
	                  }
	  
	                  override fun onError(e: Throwable) {
	  
	                  }
	  
	                  override fun onComplete() {
	  
	                  }
	  
	              };
	  
	              WbuNetEngine.ins().get(RxJavaHttpServiceDemo::class.java)
	                  .httpRx2("")
	                  .subscribe(disposable)
	              disposable
	          }
	      }
	  }
	  ```
- ## 实现
	- 通过给ViewModel添加扩展方法和属性，达到自动取消RxJava请求的目的。
		-