- # 1、WMS相关类概念
  collapsed:: true
	- ## 相关概念
		- ## [[WMS-Window]]
		- ## [[Surface]]
		- ## [[WMS-WindowManager]]
		- ## [[WindowState（在WMS中）]]
		- ## [[WMS概念]]
		- ## [[WMS-WindowToken]][[AMS-Token]]
		- ## [[窗口类型]]
		- ## [[ViewRootImpl]]
		- ## [[WindowContainer]]
	- ## 1、Activity与Window关系
	  collapsed:: true
		- Activity只负责生命周期和事件处理
		- Windowwindow表示一个窗口的概念，是所有View的直接管理者，任何视图都通过Window呈现
		- 一个Activity包含一个Window，如果Activity没有Window，那就相当于Service
		- > 通常情况下Activity展示页面只有一个window，除了分屏，弹窗等
		- AMS统一调度所有应用程序的Activity
		- WMS控制所有Window的显示与隐藏以及要显示的位置
- # 2、[[WMS启动流程]]
- # 3、[[addView到WMS流程-面试]]
- # 8、[[Activity的Window创建以及addView到WMS]]
- # 4、[[View的显示层级]]
- # 5、[[DecorView是什么？]]
- # 6、窗口WIndow显示次序分析
  collapsed:: true
	- app添加view的时候，最终会调用到 WMS.addWindow
	- 然后会创建 WindowState，在构造函数中，WindowManagerPolicy 根据窗口类型划分不同层级。层级低的展示在下边，高的展示在上边
- # 7、启动窗口window的添加流程
  collapsed:: true
	- 1、Activity启动流程执行到 ActivityStarter.startActivityUnchecked
	- 2、调用 ActivityStack.startActivityLocked
		- 2-1、把TaskRecord插入到mTaskHistory中
		- 2-2、通过ActivityRecord.createWindowContainer创建一个AppWindowToken。并且添加到DisplayContent的mTokenMap中
		- 2-3、ActivityRecord.showStartingWindow 添加第一个启动的启动Window
			- 通过PhoneWindowManager.addSplashScreen 创建一个PhoneWindow 填充内容
			- 通过WindowManager.addView 添加到WMS中
- # 参考
	- https://juejin.cn/post/6857457525764620302#heading-8