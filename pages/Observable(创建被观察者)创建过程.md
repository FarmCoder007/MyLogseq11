- ## 1、Observable.create
	- ```java
	          Observable.create(
	  
	                  // 自定义source
	                  new ObservableOnSubscribe<String>() {
	                      @Override
	                      public void subscribe(ObservableEmitter<String> emitter) throws Exception {
	                          // 发射器.onNext
	                          emitter.onNext("A");
	                      }
	          })
	  ```
	- 源码api
		- ```java
		      @CheckReturnValue
		      @SchedulerSupport(SchedulerSupport.NONE)
		      // 传入了自定义的 source
		      public static <T> Observable<T> create(ObservableOnSubscribe<T> source) {
		          // 校验是否为null
		          ObjectHelper.requireNonNull(source, "source is null");
		          // onAssembly hook函数
		         // 创建 主要在 ObservableCreate
		          return RxJavaPlugins.onAssembly(new ObservableCreate<T>(source));
		      }
		  ```
- ## 2、new ObservableCreate<T>(source)
	- [[#red]]==**创建时传入了自定义source**==
	-