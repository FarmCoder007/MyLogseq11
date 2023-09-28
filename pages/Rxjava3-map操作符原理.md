## 代码
	- ```java
	  Single.just("1")// SingleJust.map
	                  .map(new Function<String, Integer>() {
	  
	                      @Override
	                      public Integer apply(String s) throws Throwable {
	                          return Integer.parseInt(s);
	                      }
	                  }).subscribe(new SingleObserver<Integer>() {// Singmap
	                      @Override
	                      public void onSubscribe(@NonNull Disposable d) {
	  
	                      }
	  
	                      @Override
	                      public void onSuccess(@NonNull Integer s) {
	  
	                      }
	  
	                      @Override
	                      public void onError(@NonNull Throwable e) {
	  
	                      }
	                  });
	  ```
- ## 大体原理
	- 1个最简单的map流程，包含几个角色
		- 1、上游观察者以Single为例，Single.just会创建一个SingJust被观察者
		- 2、SingJust.map会创建SingMap转换对象
		- 3、订阅函数subscribe
		- 4、下游观察者SingleObserver
	- Map操作符转换的具体原理是，
		- 1、SingJust.map操作符会创建了中间桥梁对象[[#red]]==**SingMap**==它持有上游被观察者SingleJust，然后[[#red]]==**订阅下游**==自定义观察者SingleObserver
		- 2、实际订阅逻辑在SingMap的subscribeActual方法，会先创建[[#red]]==**MapSingleObserver**==包装观察者持有自定义观察者SingleObserver
		- 3、让[[#red]]==**上游SingJust订阅包装观察者MapSingleObserver**==
		- 4、上游SingJust订阅了中游SingMap内部的包装观察者，SingMap又订阅了下游的自定义观察者，实现了首尾的订阅链条
		- 5、所有SingMap相当于桥梁，接受上游发送数据，做转换处理后，交给下游观察者
	- 即使加了多个map，也是中间多个桥梁相连，让顶层订阅中间层，中间层订阅下层
- ## 具体代码原理见[[map转换过程]]