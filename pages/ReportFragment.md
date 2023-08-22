# 介绍
	- 无页面ReportFragment，监听生命周期
- # 源码
  collapsed:: true
	- ```java
	  @RestrictTo(RestrictTo.Scope.LIBRARY_GROUP_PREFIX)
	  public class ReportFragment extends android.app.Fragment {
	      private static final String REPORT_FRAGMENT_TAG = "androidx.lifecycle"
	              + ".LifecycleDispatcher.report_fragment_tag";
	  
	      public static void injectIfNeededIn(Activity activity) {
	          if (Build.VERSION.SDK_INT >= 29) {
	              // On API 29+, we can register for the correct Lifecycle callbacks directly
	              LifecycleCallbacks.registerIn(activity);
	          }
	          // Prior to API 29 and to maintain compatibility with older versions of
	          // ProcessLifecycleOwner (which may not be updated when lifecycle-runtime is updated and
	          // need to support activities that don't extend from FragmentActivity from support lib),
	          // use a framework fragment to get the correct timing of Lifecycle events
	          android.app.FragmentManager manager = activity.getFragmentManager();
	          if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
	              manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
	              // Hopefully, we are the first to make a transaction.
	              manager.executePendingTransactions();
	          }
	      }
	  
	      @SuppressWarnings("deprecation")
	      static void dispatch(@NonNull Activity activity, @NonNull Lifecycle.Event event) {
	          if (activity instanceof LifecycleRegistryOwner) {
	              ((LifecycleRegistryOwner) activity).getLifecycle().handleLifecycleEvent(event);
	              return;
	          }
	  
	          if (activity instanceof LifecycleOwner) {
	              Lifecycle lifecycle = ((LifecycleOwner) activity).getLifecycle();
	              if (lifecycle instanceof LifecycleRegistry) {
	                  ((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);
	              }
	          }
	      }
	  
	      static ReportFragment get(Activity activity) {
	          return (ReportFragment) activity.getFragmentManager().findFragmentByTag(
	                  REPORT_FRAGMENT_TAG);
	      }
	  
	      private ActivityInitializationListener mProcessListener;
	  
	      private void dispatchCreate(ActivityInitializationListener listener) {
	          if (listener != null) {
	              listener.onCreate();
	          }
	      }
	  
	      private void dispatchStart(ActivityInitializationListener listener) {
	          if (listener != null) {
	              listener.onStart();
	          }
	      }
	  
	      private void dispatchResume(ActivityInitializationListener listener) {
	          if (listener != null) {
	              listener.onResume();
	          }
	      }
	  
	      @Override
	      public void onActivityCreated(Bundle savedInstanceState) {
	          super.onActivityCreated(savedInstanceState);
	          dispatchCreate(mProcessListener);
	          dispatch(Lifecycle.Event.ON_CREATE);
	      }
	  
	      @Override
	      public void onStart() {
	          super.onStart();
	          dispatchStart(mProcessListener);
	          dispatch(Lifecycle.Event.ON_START);
	      }
	  
	      @Override
	      public void onResume() {
	          super.onResume();
	          dispatchResume(mProcessListener);
	          dispatch(Lifecycle.Event.ON_RESUME);
	      }
	  
	      @Override
	      public void onPause() {
	          super.onPause();
	          dispatch(Lifecycle.Event.ON_PAUSE);
	      }
	  
	      @Override
	      public void onStop() {
	          super.onStop();
	          dispatch(Lifecycle.Event.ON_STOP);
	      }
	  
	      @Override
	      public void onDestroy() {
	          super.onDestroy();
	          dispatch(Lifecycle.Event.ON_DESTROY);
	          // just want to be sure that we won't leak reference to an activity
	          mProcessListener = null;
	      }
	  
	      private void dispatch(@NonNull Lifecycle.Event event) {
	          if (Build.VERSION.SDK_INT < 29) {
	              // Only dispatch events from ReportFragment on API levels prior
	              // to API 29. On API 29+, this is handled by the ActivityLifecycleCallbacks
	              // added in ReportFragment.injectIfNeededIn
	              dispatch(getActivity(), event);
	          }
	      }
	  
	      void setProcessListener(ActivityInitializationListener processListener) {
	          mProcessListener = processListener;
	      }
	  
	      interface ActivityInitializationListener {
	          void onCreate();
	  
	          void onStart();
	  
	          void onResume();
	      }
	  
	      // this class isn't inlined only because we need to add a proguard rule for it (b/142778206)
	      // In addition to that registerIn method allows to avoid class verification failure,
	      // because registerActivityLifecycleCallbacks is available only since api 29.
	      @RequiresApi(29)
	      static class LifecycleCallbacks implements Application.ActivityLifecycleCallbacks {
	  
	          static void registerIn(Activity activity) {
	              activity.registerActivityLifecycleCallbacks(new LifecycleCallbacks());
	          }
	  
	          @Override
	          public void onActivityCreated(@NonNull Activity activity,
	                  @Nullable Bundle bundle) {
	          }
	  
	          @Override
	          public void onActivityPostCreated(@NonNull Activity activity,
	                  @Nullable Bundle savedInstanceState) {
	              dispatch(activity, Lifecycle.Event.ON_CREATE);
	          }
	  
	          @Override
	          public void onActivityStarted(@NonNull Activity activity) {
	          }
	  
	          @Override
	          public void onActivityPostStarted(@NonNull Activity activity) {
	              dispatch(activity, Lifecycle.Event.ON_START);
	          }
	  
	          @Override
	          public void onActivityResumed(@NonNull Activity activity) {
	          }
	  
	          @Override
	          public void onActivityPostResumed(@NonNull Activity activity) {
	              dispatch(activity, Lifecycle.Event.ON_RESUME);
	          }
	  
	          @Override
	          public void onActivityPrePaused(@NonNull Activity activity) {
	              dispatch(activity, Lifecycle.Event.ON_PAUSE);
	          }
	  
	          @Override
	          public void onActivityPaused(@NonNull Activity activity) {
	          }
	  
	          @Override
	          public void onActivityPreStopped(@NonNull Activity activity) {
	              dispatch(activity, Lifecycle.Event.ON_STOP);
	          }
	  
	          @Override
	          public void onActivityStopped(@NonNull Activity activity) {
	          }
	  
	          @Override
	          public void onActivitySaveInstanceState(@NonNull Activity activity,
	                  @NonNull Bundle bundle) {
	          }
	  
	          @Override
	          public void onActivityPreDestroyed(@NonNull Activity activity) {
	              dispatch(activity, Lifecycle.Event.ON_DESTROY);
	          }
	  
	          @Override
	          public void onActivityDestroyed(@NonNull Activity activity) {
	          }
	      }
	  }
	  
	  ```
- # 状态改变流程
	- 1、首先在ComponentActivity的onCreate 方法注入了 无页面ReportFragment
	  collapsed:: true
		- ```java
		        @Override
		        protected void onCreate(@Nullable Bundle savedInstanceState) {
		            super.onCreate(savedInstanceState);
		            // 2、注册了 无页面ReportFragment
		            ReportFragment.injectIfNeededIn(this);
		            if (mContentLayoutId != 0) {
		                setContentView(mContentLayoutId);
		            }
		        }   
		  ```
		- 看其注入方法injectIfNeededIn
			- injectIfNeededIn
			  collapsed:: true
				- ```java
				      public static void injectIfNeededIn(Activity activity) {
				          // 1、大于api29 注册 LifecycleCallbacks
				          if (Build.VERSION.SDK_INT >= 29) {
				              LifecycleCallbacks.registerIn(activity);
				          }
				          // Prior to API 29 and to maintain compatibility with older versions of
				          // ProcessLifecycleOwner (which may not be updated when lifecycle-runtime is updated and
				          // need to support activities that don't extend from FragmentActivity from support lib),
				          // use a framework fragment to get the correct timing of Lifecycle events
				          // 2、无页面Fragment 没有初始化，就通过事务添加
				          android.app.FragmentManager manager = activity.getFragmentManager();
				          if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
				              manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
				              // Hopefully, we are the first to make a transaction.
				              manager.executePendingTransactions();
				          }
				      }
				  ```
			- 1、大于api29 注册 LifecycleCallbacks。在其他地方可以只能使用[[LifecycleCallbacks]]，获取Activity生命周期方法
				- ```java
				          if (Build.VERSION.SDK_INT >= 29) {
				              LifecycleCallbacks.registerIn(activity);
				          }
				  ```
			- 2、无页面Fragment 没有初始化，就通过事务添加
	- 2、可以看到各个生命周期方法中都调用了dispatch方法，并传递了Event事件
	  collapsed:: true
		- 生命周期调用dispatch
		  collapsed:: true
			- ```java
			      @Override
			      public void onActivityCreated(Bundle savedInstanceState) {
			          super.onActivityCreated(savedInstanceState);
			          dispatchCreate(mProcessListener);
			          dispatch(Lifecycle.Event.ON_CREATE);
			      }
			  
			      @Override
			      public void onStart() {
			          super.onStart();
			          dispatchStart(mProcessListener);
			          dispatch(Lifecycle.Event.ON_START);
			      }
			  
			      @Override
			      public void onResume() {
			          super.onResume();
			          dispatchResume(mProcessListener);
			          dispatch(Lifecycle.Event.ON_RESUME);
			      }
			  
			      @Override
			      public void onPause() {
			          super.onPause();
			          dispatch(Lifecycle.Event.ON_PAUSE);
			      }
			  
			      @Override
			      public void onStop() {
			          super.onStop();
			          dispatch(Lifecycle.Event.ON_STOP);
			      }
			  
			      @Override
			      public void onDestroy() {
			          super.onDestroy();
			          dispatch(Lifecycle.Event.ON_DESTROY);
			          // just want to be sure that we won't leak reference to an activity
			          mProcessListener = null;
			      }
			  ```
	- 3、dispatch：仅在API级别之前从ReportFragment调度事件，API29及API 29+上，这由ActivityLifecycleCallbacks处理
	  collapsed:: true
		- dispatch
		  collapsed:: true
			- ```java
			      private void dispatch(@NonNull Lifecycle.Event event) {
			          if (Build.VERSION.SDK_INT < 29) {
			        	//仅在API级别之前从ReportFragment调度事件
			  
			  		//API第29条。在API 29+上，这由ActivityLifecycleCallbacks处理
			              dispatch(getActivity(), event);
			          }
			      }
			  
			      static void dispatch(@NonNull Activity activity, @NonNull Lifecycle.Event event) {
			        if (activity instanceof LifecycleRegistryOwner) {
			          ((LifecycleRegistryOwner) activity).getLifecycle().handleLifecycleEvent(event);
			          return;
			        }
			  
			        if (activity instanceof LifecycleOwner) {
			          Lifecycle lifecycle = ((LifecycleOwner) activity).getLifecycle();
			          if (lifecycle instanceof LifecycleRegistry) {
			            ((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);
			          }
			        }
			      }
			  ```
		- 主要负责获取Lifecycle，调用**handleLifecycleEvent方法**来处理生命周期Event事件。