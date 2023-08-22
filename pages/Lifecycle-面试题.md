# 1、Lifecycle作用
	- ### Lifecycle可以有效的避免内存泄漏和解决android生命周期的常见难题
		- 比如开始和停止网络连接和动画
	- ### 监听Activity和Fragment的生命周期，还**可以绑定Service和Application**的生命周期。只要引入支持相关的可选库即可
- # 2、使用
	- 1、自定义观察者，实现 DefaultLifecycleObserver接口重写需要的生命周期方法
	- 2、在Activity/Fragment里注册就可以
- # 3、原理
  collapsed:: true
	- ## 总体来说
		- 利用无页面ReportFragment，感知生命周期，计算出下一状态和事件后，经过适配器转换找到观察者的生命周期回调方法
	- ## 详细
		- 1、Activity/Fragment，创建Lifecycle子类LifecycleRegistry，并提供对外获取该实例的方法，getLifecycle()
		- 2、注册自定义观察者时，通过getLifecycle().addObserver（），将自定义观察者注册到LifecycleRegistry中
		- 3、Activity/Fragment同时添加了无页面的ReportFragment，感知生命周期，在自己的生命周期方法，计算出下一状态和事件后，经过适配器转换找到观察者的生命周期回调方法