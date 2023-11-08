# Activity的Window创建以及addView到WMS#card
	- # 一、Activity.attach中创建PhoneWindow和windowManager
		- 1、Activity启动时会调用到 ActivityThread里的performLaunchActivity()==创建Activity实例，调用attach方法==
		  collapsed:: true
			- ```java
			     private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
			      ...
			      //创建activity实例：通过类加载器创建
			  	java.lang.ClassLoader cl = appContext.getClassLoader();
			      activity = mInstrumentation.newActivity(cl, component.getClassName(), r.intent);
			      ...
			      //调用﻿Activity的attach方法--关联上下文环境变量
			    	activity.attach(appContext, this, getInstrumentation(), r.token,
			                          r.ident, app, r.intent, r.activityInfo, title, r.parent,
			                          r.embeddedID, r.lastNonConfigurationInstances, config,
			                          r.referrer, r.voiceInteractor, window, r.configCallback);    
			  	...
			  }
			  ```
		- 2、[[#red]]==**activity.attach，创建PhoneWIndow 和 设置WindowManager**==
		  collapsed:: true
			- ```java
			   //实例化window，就是Window的唯一实现PhoneWindow
			          mWindow = new PhoneWindow(this, window, activityConfigCallback);
			          ...
			          //把activity作为回调接口传入window，这样window从外界接受的状态变化都会交给activity
			          //例如：dispatchTouchEvent、onAttachedToWindow、onDetachedFromWindow
			          mWindow.setCallback(this);
			          ...
			          //设置windowManager，实际就是WindowManagerImpl的实例，在activity中getWindowManager()获取的就是这个实例
			          mWindow.setWindowManager(
			                  (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
			                  mToken, mComponent.flattenToString(),
			                  (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
			          ...
			          mWindowManager = mWindow.getWindowManager();
			  ```
	- # 二、DecorView的创建
		- 1、根据Activity的启动流程，得知Instrumentation创建Activity后会调用callActivityCreated方法。最终Activity的==**onCreate（）生命周期回调**==
		  collapsed:: true
		- 2、Activity的 onCreate()中 会调用setContentView，实际调用的是PhoneWindow.setContentView
		  collapsed:: true
			- ```java
			    public void setContentView(@LayoutRes int layoutResID) {
			          getWindow().setContentView(layoutResID);
			          initWindowDecorActionBar();
			      }
			  ```
		- 2、PhoneWindow的setContentView中，如果视图容器mContentParent（FrameLayout）为空就调用installDecor
		  collapsed:: true
			- ```java
			  public void setContentView(int layoutResID) {
			          // mContentParent为空，就调installDecor()，猜想installDecor()里面创建了mContentParent。且从名字看出mContentParent就是内容视图的容器
			          if (mContentParent == null) {
			              installDecor();
			          } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
			              mContentParent.removeAllViews();
			          }
			  
			          if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
			              final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
			                      getContext());
			              transitionTo(newScene);
			          } else {
			          	//这里看到，确实把我们的视图加载到mContentParent了
			              mLayoutInflater.inflate(layoutResID, mContentParent);
			          }
			          ...
			      }
			  ```
		- 3、installDecor()-->A、创建DecorVIew，B、将设置的布局视图添加到DecorView的 FrameLayout中
		  collapsed:: true
			- ```java
			  private void installDecor() {
			  	if (mDecor == null) {
			  			//创建mDecor
			              mDecor = generateDecor(-1);
			              mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
			              mDecor.setIsRootNamespace(true);
			              if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
			                  mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
			              }
			          } else {
			              mDecor.setWindow(this);
			          }
			          if (mContentParent == null) {
			          	//创建mContentParent
			              mContentParent = generateLayout(mDecor);
			     ...
			  }
			  ```
	- # 三、DecorVIew -> addView到WMS通知渲染
		- 参考[[addView到WMS流程]]
		- 1、在ActivityThred的[[#green]]==**handleResumeActivity**==()中，调用activity.makeVisible 内部判断 window是否添加
		- 2、未添加的话，会调用[[#red]]==WindowManagerGlobal.addView==添加mDecorview
		  collapsed:: true
			- 拿到windowManager（父类 viewManager）添加mDecor
		- 2、addView方法会创建ViewRootImpl实例，调用ViewRootImpl.setView方法
			- ## 1、ViewRootImpl的setView里先调用了requestLayout触发渲染流程，
		- 2、setView里 通过 Session.addToDisplay  和  WMS  IPC通信，传递view相关参数
		- 4、WMS.addWindow创建 WindowState，以及通知底层绘制