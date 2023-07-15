- # 一、看activity.attach
  collapsed:: true
	- ## 1、Activity启动会调到ActivityThread的performLaunchActivity，通过Instrumentation创建Activity,执行Activity的attach
		- ```java
		  private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
		    
		    
		                activity = mInstrumentation.newActivity(
		                      cl, component.getClassName(), r.intent);
		      Application app = r.packageInfo.makeApplication(false, mInstrumentation);
		     activity.attach(appContext, this, getInstrumentation(), r.token,
		                          r.ident, app, r.intent, r.activityInfo, title, r.parent,
		                          r.embeddedID, r.lastNonConfigurationInstances, config,
		                          r.referrer, r.voiceInteractor, window, r.configCallback,
		                          r.assistToken);
		  
		  }
		  ```
		- attach传了token（IBinder），来自ActivityClientRecord类。这个token其实来源于AMS的ActivityRecord中的token详见[[AMS-Token]]
		- ==**结论：**==Window里的token就是对应的activity的弱引用
	- ## 2、attach具体操作：创建了phoneWindow和WindowManager对象
		- activity.attach
		  collapsed:: true
			- ```java
			      @UnsupportedAppUsage
			      final void attach(Context context, ActivityThread aThread,
			              Instrumentation instr, IBinder token, int ident,
			              Application application, Intent intent, ActivityInfo info,
			              CharSequence title, Activity parent, String id,
			              NonConfigurationInstances lastNonConfigurationInstances,
			              Configuration config, String referrer, IVoiceInteractor voiceInteractor,
			              Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken) {
			          attachBaseContext(context);
			  
			          mFragments.attachHost(null /*parent*/);
			          // 1创建PhoneWindow
			          mWindow = new PhoneWindow(this, window, activityConfigCallback);
			          mWindow.setWindowControllerCallback(this);
			          mWindow.setCallback(this);
			          mWindow.setOnWindowDismissedCallback(this);
			          mWindow.getLayoutInflater().setPrivateFactory(this);
			          if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
			              mWindow.setSoftInputMode(info.softInputMode);
			          }
			          if (info.uiOptions != 0) {
			              mWindow.setUiOptions(info.uiOptions);
			          }
			          mUiThread = Thread.currentThread();
			  
			          mMainThread = aThread;
			          mInstrumentation = instr;
			          mToken = token;
			          mAssistToken = assistToken;
			          mIdent = ident;
			          mApplication = application;
			          mIntent = intent;
			          mReferrer = referrer;
			          mComponent = intent.getComponent();
			          mActivityInfo = info;
			          mTitle = title;
			          mParent = parent;
			          mEmbeddedID = id;
			          mLastNonConfigurationInstances = lastNonConfigurationInstances;
			          if (voiceInteractor != null) {
			              if (lastNonConfigurationInstances != null) {
			                  mVoiceInteractor = lastNonConfigurationInstances.voiceInteractor;
			              } else {
			                  mVoiceInteractor = new VoiceInteractor(voiceInteractor, this, this,
			                          Looper.myLooper());
			              }
			          }
			          // 2 设置windowmanager
			          mWindow.setWindowManager(
			                  (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
			                  mToken, mComponent.flattenToString(),
			                  (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
			          if (mParent != null) {
			              mWindow.setContainer(mParent.getWindow());
			          }
			          mWindowManager = mWindow.getWindowManager();
			          mCurrentConfig = config;
			  
			          mWindow.setColorMode(info.colorMode);
			  
			          setAutofillOptions(application.getAutofillOptions());
			          setContentCaptureOptions(application.getContentCaptureOptions());
			      }
			  ```
		- 1、创建了phoneWindow（WIndow子类）
		- 2、通过父类方法setWindowManager,创建windowmanager对象，WindowManager最终实现是在WindowManagerImpl
			- Window.setWindowManager
				- WindowManagerImpl.createLocalWindowManager
					- new WindowManagerImpl，里边添加view实际交给了WindowManagerGrobal（单例）
			-
		- 3、将之前创建的WindowManager对象保存在mWindowManager对象中
			- mWindowManager = mWindow.getWindowManager
- # 二、看添加view的过程
	- ## 1、activity resume-> view添加到WMS -> SurfaceFlilger
	- ## 2、startActivity 启动activity时 startWindow 启动窗口
	- ## [[WMS中添加view前提知识]]
	- ## [[addView到WMS流程]]
		-
-