- ## 1、入口SystemServer的Run(）方法，调用startOtherServices();
- ## 2、startOtherServices创建WMS，执行main方法
  collapsed:: true
	- ```java
	  private void startOtherServices() {
	    wm = WindowManagerService.main(context, inputManager, !mFirstBoot, mOnlyCore,
	                                   new PhoneWindowManager(),
	                                   mActivityManagerService.mActivityTaskManager);
	    ServiceManager.addService(Context.WINDOW_SERVICE, wm, /* allowIsolated= */ false,
	                              DUMP_FLAG_PRIORITY_CRITICAL | DUMP_FLAG_PROTO);
	  }
	  ```
- ## 3、WindowManagerService.main()初始化的时候，注意区分不同线程
  id:: 64acd3db-b1ff-4eb0-a4b8-d62595afb67a
	- 1、Display
	- 2、ui 线程
	- 3、systemServer主线程
	- ```java
	      private static WindowManagerService sInstance;
	      public static WindowManagerService main(final Context context, final InputManagerService im,
	              final boolean showBootMsgs, final boolean onlyCore, WindowManagerPolicy policy,
	              ActivityTaskManagerService atm, TransactionFactory transactionFactory) {
	          DisplayThread.getHandler().runWithScissors(() ->
	                  sInstance = new WindowManagerService(context, im, showBootMsgs, onlyCore, policy,
	                          atm, transactionFactory), 0);
	          return sInstance;
	      }
	  ```
	- 1、首先，main运行在systemServer主线程的
	- 2、在main方法里里，通过跨线程通信，wms是在Display线程里通过DisplayThread.getHandler().runWithScissors 同步等待WMS初始化成功
		- 详细见[[Handler().runWithScissors 同步等待]]
		-
- ## 4、WMS构造函数的初始化工作
	- api29源码
	  collapsed:: true
		- ```java
		      private WindowManagerService(Context context, InputManagerService inputManager,
		              boolean showBootMsgs, boolean onlyCore, WindowManagerPolicy policy,
		              ActivityTaskManagerService atm, TransactionFactory transactionFactory) {
		          installLock(this, INDEX_WINDOW);
		          mGlobalLock = atm.getGlobalLock();
		          mAtmService = atm;
		          mContext = context;
		          mAllowBootMessages = showBootMsgs;
		          mOnlyCore = onlyCore;
		          mLimitedAlphaCompositing = context.getResources().getBoolean(
		                  com.android.internal.R.bool.config_sf_limitedAlpha);
		          mHasPermanentDpad = context.getResources().getBoolean(
		                  com.android.internal.R.bool.config_hasPermanentDpad);
		          mInTouchMode = context.getResources().getBoolean(
		                  com.android.internal.R.bool.config_defaultInTouchMode);
		          mDrawLockTimeoutMillis = context.getResources().getInteger(
		                  com.android.internal.R.integer.config_drawLockTimeoutMillis);
		          mAllowAnimationsInLowPowerMode = context.getResources().getBoolean(
		                  com.android.internal.R.bool.config_allowAnimationsInLowPowerMode);
		          mMaxUiWidth = context.getResources().getInteger(
		                  com.android.internal.R.integer.config_maxUiWidth);
		          mDisableTransitionAnimation = context.getResources().getBoolean(
		                  com.android.internal.R.bool.config_disableTransitionAnimation);
		          mPerDisplayFocusEnabled = context.getResources().getBoolean(
		                  com.android.internal.R.bool.config_perDisplayFocusEnabled);
		          mLowRamTaskSnapshotsAndRecents = context.getResources().getBoolean(
		                  com.android.internal.R.bool.config_lowRamTaskSnapshotsAndRecents);
		          mInputManager = inputManager; // Must be before createDisplayContentLocked.
		          mDisplayManagerInternal = LocalServices.getService(DisplayManagerInternal.class);
		          mDisplayWindowSettings = new DisplayWindowSettings(this);
		  
		          mTransactionFactory = transactionFactory;
		          mTransaction = mTransactionFactory.make();
		          mPolicy = policy;
		          // 1初始化 动画管理 WindowAnimator
		          mAnimator = new WindowAnimator(this);
		          mRoot = new RootWindowContainer(this);
		  
		          mWindowPlacerLocked = new WindowSurfacePlacer(this);
		          mTaskSnapshotController = new TaskSnapshotController(this);
		  
		          mWindowTracing = WindowTracing.createDefaultAndStartLooper(this,
		                  Choreographer.getInstance());
		  
		          LocalServices.addService(WindowManagerPolicy.class, mPolicy);
		  
		          mDisplayManager = (DisplayManager)context.getSystemService(Context.DISPLAY_SERVICE);
		  
		          mKeyguardDisableHandler = KeyguardDisableHandler.create(mContext, mPolicy, mH);
		  
		          mPowerManager = (PowerManager)context.getSystemService(Context.POWER_SERVICE);
		          mPowerManagerInternal = LocalServices.getService(PowerManagerInternal.class);
		  
		          if (mPowerManagerInternal != null) {
		              mPowerManagerInternal.registerLowPowerModeObserver(
		                      new PowerManagerInternal.LowPowerModeListener() {
		                  @Override
		                  public int getServiceType() {
		                      return ServiceType.ANIMATION;
		                  }
		  
		                  @Override
		                  public void onLowPowerModeChanged(PowerSaveState result) {
		                      synchronized (mGlobalLock) {
		                          final boolean enabled = result.batterySaverEnabled;
		                          if (mAnimationsDisabled != enabled && !mAllowAnimationsInLowPowerMode) {
		                              mAnimationsDisabled = enabled;
		                              dispatchNewAnimatorScaleLocked(null);
		                          }
		                      }
		                  }
		              });
		              mAnimationsDisabled = mPowerManagerInternal
		                      .getLowPowerState(ServiceType.ANIMATION).batterySaverEnabled;
		          }
		          mScreenFrozenLock = mPowerManager.newWakeLock(
		                  PowerManager.PARTIAL_WAKE_LOCK, "SCREEN_FROZEN");
		          mScreenFrozenLock.setReferenceCounted(false);
		          // 获取AMS
		          mActivityManager = ActivityManager.getService();
		          mActivityTaskManager = ActivityTaskManager.getService();
		          mAmInternal = LocalServices.getService(ActivityManagerInternal.class);
		          mAtmInternal = LocalServices.getService(ActivityTaskManagerInternal.class);
		          mAppOps = (AppOpsManager)context.getSystemService(Context.APP_OPS_SERVICE);
		          AppOpsManager.OnOpChangedInternalListener opListener =
		                  new AppOpsManager.OnOpChangedInternalListener() {
		                      @Override public void onOpChanged(int op, String packageName) {
		                          updateAppOpsState();
		                      }
		                  };
		          mAppOps.startWatchingMode(OP_SYSTEM_ALERT_WINDOW, null, opListener);
		          mAppOps.startWatchingMode(AppOpsManager.OP_TOAST_WINDOW, null, opListener);
		  
		          mPmInternal = LocalServices.getService(PackageManagerInternal.class);
		          final IntentFilter suspendPackagesFilter = new IntentFilter();
		          suspendPackagesFilter.addAction(Intent.ACTION_PACKAGES_SUSPENDED);
		          suspendPackagesFilter.addAction(Intent.ACTION_PACKAGES_UNSUSPENDED);
		          context.registerReceiverAsUser(new BroadcastReceiver() {
		              @Override
		              public void onReceive(Context context, Intent intent) {
		                  final String[] affectedPackages =
		                          intent.getStringArrayExtra(Intent.EXTRA_CHANGED_PACKAGE_LIST);
		                  final boolean suspended =
		                          Intent.ACTION_PACKAGES_SUSPENDED.equals(intent.getAction());
		                  updateHiddenWhileSuspendedState(new ArraySet<>(Arrays.asList(affectedPackages)),
		                          suspended);
		              }
		          }, UserHandle.ALL, suspendPackagesFilter, null, null);
		  
		          final ContentResolver resolver = context.getContentResolver();
		          // Get persisted window scale setting
		          mWindowAnimationScaleSetting = Settings.Global.getFloat(resolver,
		                  Settings.Global.WINDOW_ANIMATION_SCALE, mWindowAnimationScaleSetting);
		          mTransitionAnimationScaleSetting = Settings.Global.getFloat(resolver,
		                  Settings.Global.TRANSITION_ANIMATION_SCALE,
		                  context.getResources().getFloat(
		                          R.dimen.config_appTransitionAnimationDurationScaleDefault));
		  
		          setAnimatorDurationScale(Settings.Global.getFloat(resolver,
		                  Settings.Global.ANIMATOR_DURATION_SCALE, mAnimatorDurationScaleSetting));
		  
		          mForceDesktopModeOnExternalDisplays = Settings.Global.getInt(resolver,
		                  DEVELOPMENT_FORCE_DESKTOP_MODE_ON_EXTERNAL_DISPLAYS, 0) != 0;
		  
		          IntentFilter filter = new IntentFilter();
		          // Track changes to DevicePolicyManager state so we can enable/disable keyguard.
		          filter.addAction(ACTION_DEVICE_POLICY_MANAGER_STATE_CHANGED);
		          mContext.registerReceiverAsUser(mBroadcastReceiver, UserHandle.ALL, filter, null, null);
		  
		          mLatencyTracker = LatencyTracker.getInstance(context);
		  
		          mSettingsObserver = new SettingsObserver();
		  
		          mHoldingScreenWakeLock = mPowerManager.newWakeLock(
		                  PowerManager.SCREEN_BRIGHT_WAKE_LOCK | PowerManager.ON_AFTER_RELEASE, TAG_WM);
		          mHoldingScreenWakeLock.setReferenceCounted(false);
		  
		          mSurfaceAnimationRunner = new SurfaceAnimationRunner(mPowerManagerInternal);
		  
		          mAllowTheaterModeWakeFromLayout = context.getResources().getBoolean(
		                  com.android.internal.R.bool.config_allowTheaterModeWakeFromWindowLayout);
		  
		          mTaskPositioningController = new TaskPositioningController(
		                  this, mInputManager, mActivityTaskManager, mH.getLooper());
		          mDragDropController = new DragDropController(this, mH.getLooper());
		  
		          mSystemGestureExclusionLimitDp = Math.max(MIN_GESTURE_EXCLUSION_LIMIT_DP,
		                  DeviceConfig.getInt(DeviceConfig.NAMESPACE_WINDOW_MANAGER,
		                          KEY_SYSTEM_GESTURE_EXCLUSION_LIMIT_DP, 0));
		          mSystemGestureExcludedByPreQStickyImmersive =
		                  DeviceConfig.getBoolean(DeviceConfig.NAMESPACE_WINDOW_MANAGER,
		                          KEY_SYSTEM_GESTURES_EXCLUDED_BY_PRE_Q_STICKY_IMMERSIVE, false);
		          DeviceConfig.addOnPropertiesChangedListener(DeviceConfig.NAMESPACE_WINDOW_MANAGER,
		                  new HandlerExecutor(mH), properties -> {
		                      synchronized (mGlobalLock) {
		                          final int exclusionLimitDp = Math.max(MIN_GESTURE_EXCLUSION_LIMIT_DP,
		                                  properties.getInt(KEY_SYSTEM_GESTURE_EXCLUSION_LIMIT_DP, 0));
		                          final boolean excludedByPreQSticky = DeviceConfig.getBoolean(
		                                  DeviceConfig.NAMESPACE_WINDOW_MANAGER,
		                                  KEY_SYSTEM_GESTURES_EXCLUDED_BY_PRE_Q_STICKY_IMMERSIVE, false);
		                          if (mSystemGestureExcludedByPreQStickyImmersive != excludedByPreQSticky
		                                  || mSystemGestureExclusionLimitDp != exclusionLimitDp) {
		                              mSystemGestureExclusionLimitDp = exclusionLimitDp;
		                              mSystemGestureExcludedByPreQStickyImmersive = excludedByPreQSticky;
		                              mRoot.forAllDisplays(DisplayContent::updateSystemGestureExclusionLimit);
		                          }
		                      }
		                  });
		  
		          LocalServices.addService(WindowManagerInternal.class, new LocalService());
		      }
		  ```
	- 作用：初始化WMS用到的东西，和一些服务
- ## 5、WMS初始化完成后，把WMS设置给AMS，进行关联
	- SystemServer.startOtherServices
		- ```java
		  mActivityManagerService.setWindowManager(wm);
		  ```
- ## 6、然后执行  wms.onInitReady();
  collapsed:: true
	- initPolicy->UiThread.runWithScissors
		- ```java
		      private void initPolicy() {
		          UiThread.getHandler().runWithScissors(new Runnable() {
		              @Override
		              public void run() {
		                  WindowManagerPolicyThread.set(Thread.currentThread(), Looper.myLooper());
		                  mPolicy.init(mContext, WindowManagerService.this, WindowManagerService.this);
		              }
		          }, 0);
		      }
		  ```
	-
- ## 7、wm.displayReady()初始化显示尺寸信息，结束后WMS会根据AMS进行一次configure
- ## 8、wm.systemReady()  直接调用mPolicy的systemready方法
-
- ## 总结
	- WMS启动涉及3个线程，SystemServer,android.display,android.ui
	- 其中WMS.H.HandlerMessage运行在android.display线程中
	- ## 三个关键步骤
		- 1、创建WMS对象
		- 2、初始化显示信息
		- 3、处理systemready通知