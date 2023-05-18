- 本文会由浅入深的为大家介绍 Jetpack 的成员之一：LiveData。内容包括 LiveData 的简介、基本使用，原理分析（如何感知宿主生命周期变化，消息分发流程等），在文章末尾会比较一下 LiveData 和 Flow 的一些区别。
- ![image.png](../assets/image_1684422084156_0.png)
- # 1. LiveData 简介
  collapsed:: true
	- ## 1.1 LivaData 是什么？
	  collapsed:: true
		- LiveData 是一种具有生命周期（如 Activity、Fragment 或 Service）感知能力的、可观察的数据存储器类。LiveData 仅更新处于活跃生命周期状态的应用组件观察者。
		- ![image.png](../assets/image_1684422110817_0.png)
	- ## 1.2 LivaData 的优势
	  collapsed:: true
		- LiveData 能确保 UI 和数据状态相符
		  LiveData 遵循观察者模式。当底层数据发生变化时，LiveData 会通知 Observer 对象来更新界面。
		- 不会发生内存泄漏
		  观察者和 Lifecycle 对象绑定，能在销毁时自动解除注册。
		- 不会给已经停止的 Activity 发送事件
		  如果观察者处于非活跃状态，LiveData 不会再发送任何事件给这些 Observer 对象。
		- 不再需要手动处理生命周期
		  UI 组件仅仅需要对相关数据进行观察，LiveData 自动处理生命周期状态改变后，需要处理的代码。
		- 数据始终保持最新状态
		  一个非活跃的组件进入到活跃状态后，会立即获取到最新的数据，不用担心数据问题。
		- LiveData 在横竖屏切换等 Configuration 改变时，也能保证获取到最新数据
		  例如 Acitivty、Fragment 因为屏幕选装导致重建, 能立即接收到最新的数据。
		- LiveData 能资源共享
		  如果将 LiveData 对象扩展，用单例模式将系统服务进行包裹。这些服务就可以在 APP 中共享。
- # 2 LiveData 的使用
	- ## 2.1 MutableLiveData 的使用
		-