- ViewModel的RxJava扩展
- 在ViewModel中我们可以使用RxJava进行网络请求，接口如下：
  collapsed:: true
	- ```
	  interface RxJavaHttpServiceDemo {
	      fun httpRx1(infoId: String): rx.Observable<API<String>>
	  
	      fun httpRx2(infoId: String): io.reactivex.Observable<API<String>>
	  }
	  ```