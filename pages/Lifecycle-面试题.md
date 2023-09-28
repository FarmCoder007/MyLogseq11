# 1、Lifecycle作用
	- ### 1、Lifecycle可以有效的==**避免内存泄漏**==、解决android==**生命周期的常见难题**==
		- 比如开始和停止网络连接
		- 控制页面动画的启动与停止：
	- ### 3、[[#red]]==**监听Activity和Fragment的生命周期**==，还**可以绑定Service和Application**的生命周期。只要引入支持相关的可选库即可
- # 2、使用
	- 1、自定义观察者，实现 DefaultLifecycleObserver接口重写需要的生命周期方法
		- ```java
		  class MyObserver : DefaultLifecycleObserver {
		      override fun onResume(owner: LifecycleOwner) {
		          // 执行业务逻辑
		      }
		  
		      override fun onPause(owner: LifecycleOwner) {
		          // 执行业务逻辑
		      }
		      ...
		  }
		  
		  ```
	- 2、在Activity/Fragment (生命周期被观察者LifecycleOwner)里注册就可以
		- ```kotlin
		  class MainActivity: AppCompatActivity{
		  	@Override
		  	fun onCreate(){
		  		getLifecycle().addObserver(MyObserver())
		  	}
		  }
		  
		  ```
		- LifecycleOwner接口只有一个方法getLifecycle()，返回的是Lifecycle对象。实现了这个接口，就表示这个类是具有生命周期的。
		- [[#red]]==**Activity和Fragment，已经默认实现了该接口，成为被观察者**==
- # 3、原理
	- ## 总体来说
		- 利用无页面ReportFragment，感知生命周期，计算出下一状态和事件后，经过适配器转换找到观察者的生命周期回调方法
	- ## 详细
		- 1、Activity/Fragment，会创建Lifecycle子类LifecycleRegistry，并提供对外获取该实例的方法，[[#red]]==**getLifecycle**==()
		- 2、我们通过getLifecycle().addObserver（），将自定义观察者注册到LifecycleRegistry中
		- 3、Activity/Fragment同时添加了空白的ReportFragment，感知生命周期，在自己的生命周期方法，计算出[[#red]]==**下一状态和事件后**==，经过适配器转换后，回调观察者的对应生命周期方法