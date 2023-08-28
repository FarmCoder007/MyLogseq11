## Activity 里 有两个成员变量  PhoneWindow  和  WindowManagerImpl
- ## 1、activity 在 onCreate()里调用了setContentView()  实际调用 PhoneWindow的 setContentView（）
	- 【activity 通过 phoneWindow  与View 链接的】
- ## 2、PhoneWindow在setContentView（）里创建了DecorView
  collapsed:: true
	- 【PhoneWindow 和 View 通过DecorView联系起来的  】
	- [[#red]]==**并添加默认布局**==：一个LinearLayout 里包含一个 标题栏 和 Framelayout（id为Content）
	- ### 3、DecorVIew里  有个默认布局：
	  collapsed:: true
		- 我们 从 SetContentView传入的布局文件通过inflat  成view , 然后添加到这个id为content的 Framelayout里的
		- ![绘制流程.png](../assets/绘制流程_1693041027183_0.png)
- ## 3、通过LayoutInflater.inflate实例化xml里的布局，实际调用到createaViewFromTag 反射创建view实例的，生成布局参数，通过addView方法添加到Framlayout上
- ## 4、addView调用requestLayout【view在调用 requestLayout时  是逐层向上调用的，直到ViewRootImpl.requestlayout】
- ## 5、ViewRootImpl.requestlayout中调用scheduleTraversals，最终调用performTraversals
- ## 6、ViewRootImpl，在performTraversals()里 执行performMeasure()  performLayout   performDraw 【然后逐层向下测量绘制】
-
-
- >ViewRootImpl         是在WindowmanagerGlobal里创建的
- >DecorView 的父view 是 ViewRootImpl
-
- # [[View的requestLayout到performTraversals]]