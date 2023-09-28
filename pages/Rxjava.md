# 中文文档
	- https://mcxiaoke.gitbooks.io/rxdocs/content/topics/How-To-Use-RxJava.html
- ## 依赖
	- ```java
	      implementation "io.reactivex.rxjava3:rxjava:3.1.6"
	      implementation 'io.reactivex.rxjava3:rxandroid:3.0.2'
	  ```
- # 一、核心思想
	- rx为响应式编程的思维,
		- 有一个起点和终点，起点开始流向我们的事件，把事件流向终点，只不过在流向的过程中，可以增加拦截，拦截时可以对“事件进行改变”，终点只关心它上一个拦截
	- 起点为开始事件
	- 终点为observer,观察者
- # 二、Observer观察者
  collapsed:: true
	- 观察者
		- ```java
		  .subscribe(new Observer<String>() {
		              // 订阅开始
		              @Override
		              public void onSubscribe(Disposable d) {
		                  disposable = d;
		              }
		              // 拿到事件
		              @Override
		              public void onNext(String s) {}
		              // 错误事件
		              @Override
		              public void onError(Throwable e) {}
		              // 完成事件
		              @Override
		              public void onComplete() {}
		          });
		  ```
	- 简化版本的观察者
		- ```java
		      public void getProjectAction(View view) {
		          // 获取网络API
		          api.getProject()
		                  .subscribeOn(Schedulers.io()) // 上面 异步
		                  .observeOn(AndroidSchedulers.mainThread()) // 下面 主线程
		                  .subscribe(new Consumer<ProjectBean>() {
		                      @Override
		                      public void accept(ProjectBean projectBean) throws Exception {
		                          Log.d(TAG, "accept: " + projectBean); // UI 可以做事情
		                      }
		                  });
		      }
		  ```
- # 三、变换操作符map，从中间进行数据转换
- # 四、[[rxjava操作示例]]
- # 五、[[rxjava与Retrofit封装]]
- # 六、[[网络嵌套之flatMap]]
- # 七、[[doOnNext()]] 线程频繁切换
- # 八、[[RXjava-全局hook点onAssembly]]
	- # [[rxjava2.0-hookScheduler]]
- # 九、[[rxjava里的观察者模式（非常说的观察者）-rx2.0]]
- # 十、[[Map源码分析-rx2.0]]
- # 十一、[[背压]]
- # 十二、[[Rxjava2.0线程切换]]
-
-
- # 十三、[[Rxjava3原理解析]]
- # 十四、[[订阅发生时，流转过程]]
- # [[Rxjava面试题]]
- # 资料
	- ## 入门相关的资料与博客
		- https://www.jianshu.com/p/cd3557b1a474
		  https://www.cnblogs.com/lyysz/p/6344507.html
		  https://www.cnblogs.com/liushilin/p/7058302.html
		  https://www.jb51.net/article/92309.htm
		  https://zhuanlan.zhihu.com/p/31413825
	- ## 基础相关的资料与博客
	  collapsed:: true
		- https://github.com/ReactiveX/RxJava
		- [[RxJava全套使用完整版]]
		- [RxJava操作符大全.xmind](../assets/RxJava操作符大全_1690193315396_0.xmind)
	- ## RxJava2.0 与 RxJava3.0差异化
	  collapsed:: true
		- https://blog.csdn.net/weixin_45258969/article/details/95386872?utm_medium=distribute.down_relevant_right.none-task-blog-BlogCommendFromBaidu-1.nonecase&depth_1-utm_source=distribute.down_relevant_right.none-task-blog-BlogCommendFromBaidu-1.nonecase