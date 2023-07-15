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
	- 1、Activity的setContentView会调用到PhoneWindow.setContentView
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
	- 3、installDecor()
	  collapsed:: true
		- 1、创建DecorVIew
		- 2、将设置的布局视图添加到DecorView的 FrameLayout中
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
- # 三、在ActivityThred的handleResumeActivity()中
	- 参考[[addView到WMS流程]]
	- 1、activity.makeVisible 会判断 如果window还没有添加
		- 会调用[[#red]]==ViewManager.addView==添加mDecorview
			- 拿到windowManager（父类 viewManager）添加mDecor
	- 2、最终会调用到WindowManagerGlobal.addView  创建ViewRootImpl实例，调用root.setView方法
	- 3、最终通过 Session.addToDisplay  和  WMS  IPC通信，将view相关参数传递
	- 4、创建 WindowState，以及通知底层绘制