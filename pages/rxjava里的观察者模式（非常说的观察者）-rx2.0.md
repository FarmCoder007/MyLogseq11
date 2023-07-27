title:: rxjava里的观察者模式（非常说的观察者）-rx2.0

- ## Observable和Observer的订阅过程
	- 代码
	  collapsed:: true
		- ```java
		  
		  /**
		   * TODO RxJava的观察者模式
		   * 1：Observer 源码看看
		   * 2：Observable创建过程 源码分析
		   * 3：subscribe订阅过程 源码分析
		   */
		  public class SourceActivity2 extends AppCompatActivity {
		  
		      @Override
		      protected void onCreate(Bundle savedInstanceState) {
		          super.onCreate(savedInstanceState);
		  
		          // 结论： new ObservableCreate() {source == 自定义source}
		          // 2：Observable创建过程 源码分析
		          Observable.create(
		                  // 自定义source
		                  new ObservableOnSubscribe<String>() {
		                      @Override
		                      public void subscribe(ObservableEmitter<String> emitter) throws Exception {
		                          // 发射器.onNext
		                          emitter.onNext("A");
		                      }
		          })
		          // 3：subscribe订阅过程 源码分析
		          // ObservableCreate. subscribe
		          .subscribe(
		                  // 自定义观察者
		                  // 1：Observer 源码看看
		                  new Observer<String>() {
		                      @Override
		                      public void onSubscribe(Disposable d) {
		  
		                      }
		  
		                      @Override
		                      public void onNext(String s) {
		  
		                      }
		  
		                      @Override
		                      public void onError(Throwable e) {
		  
		                      }
		  
		                      @Override
		                      public void onComplete() {
		  
		                      }
		                  });
		      }
		  }
		  
		  ```
	- # 1：[[Observer（创建观察者）]]
	- # 2：[[Observable(创建被观察者)创建过程]]
	- # 3：[[subscribe订阅过程 源码分析]]
- ## [[标准观察者和rxJava观察者的区别？]]
-