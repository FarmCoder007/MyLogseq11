title:: subscribeOn (Schedulers.io())怎么切线程？

- ```java
          Single.just(1) //SingJust
                  .subscribeOn(Schedulers.io()) // SingleSubscribeOn
                  .subscribe(new SingleObserver<Integer>() {
                      @Override
                      public void onSubscribe(@NonNull Disposable d) {
                          d.dispose();
                      }
  
                      @Override
                      public void onSuccess(@NonNull Integer s) {
  
                      }
  
                      @Override
                      public void onError(@NonNull Throwable e) {
  
                      }
                  });
  ```
- ## subscribeOn（他会在订阅发生前切线程）通俗解释： 面试答这个：
	- 1、Schedulers.io：创建了一个IoScheduler的线程池
	- 2、SingleJust.subscribeOn（Schedulers.io()）会创建SingleSubscribeOn中间桥梁，内部传入IoScheduler线程池，和上游被观察者Singlejust
	- 3、当调用subscribe订阅方法时，相当于SingleSubscribeOn中间桥梁订阅 下游观察者
	- 4、则看SingleSubscribeOn实际订阅方法subscribeActual：创建了一个包装观察者SubscribeOnObserver，同时也是Runnable。让线程池执行这个任务
	- 5、run方法中让上游SingleJust订阅 这个包装观察者。此时已经切换到了子线程。上游发送消息和观察者回调都在子线程了
	-
	- 取消订阅的时候：下游观察者调用我的 disposeable.dispose 方法     1 我调用上游的取消订阅方法2 同时取消我内部的线程任务
- # 注意： 多次  subscribeOn 调用 只有写在最上一行的才对订阅 起作用
- [[SubScribeOn给上面代码分配线程-面试]]