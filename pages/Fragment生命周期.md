# 一、图
collapsed:: true
	- ![fragment_lifecycle.png](../assets/fragment_lifecycle_1689238053290_0.png){:height 855, :width 317}
- ## 1、**onAttach(Context)**
  collapsed:: true
	- onAttach() 是一个 Fragment 和它的 Context 关联时第一个调用的方法，这里我们可以获得对应的Context 或者 Activity ，可以看到这里拿到的 Activity 是 mHost.getActivity() ，后面我们介绍FragmentManager 时介绍这个方法。
	- ```java
	  @CallSuper
	  public void onAttach(Context context) {
	      mCalled = true;
	      final Activity hostActivity = mHost == null ? null :
	      mHost.getActivity();
	      if (hostActivity != null) {
	          mCalled = false;
	          onAttach(hostActivity);
	      }
	  }
	  
	  @Deprecated
	  @CallSuper
	  public void onAttach(Activity activity) {
	  	mCalled = true;
	  }
	  ```
- ## 2、**onCreate(Bundle)**
  collapsed:: true
	- onCreate() 在 onAttach() 后调用，用于做一些初始化操作。
	- 需要注意的是，Fragment 的 onCreate() 调用时关联的 Activity 可能还没创建好，所以这里不要有依赖外部 Activity 布局的操作。如果有依赖 Activity 的操作，可以放在 onActivityCreate() 中。从上面的代码还可以看到，如果是从旧状态中恢复，会执行子 Fragment 状态的恢复，此外还在onCreate() 中调用了子 Fragment 管理者的创建。
	- ```java
	  public void onCreate(@Nullable Bundle savedInstanceState) {
	      mCalled = true;
	      restoreChildFragmentState(savedInstanceState);
	      if (mChildFragmentManager != null
	                          && !mChildFragmentManager.isStateAtLeast(Fragment.CREATED)) {
	          mChildFragmentManager.dispatchCreate();
	      }
	  }
	  void restoreChildFragmentState(@Nullable Bundle savedInstanceState) {
	      if (savedInstanceState != null) {
	          Parcelable p = savedInstanceState.getParcelable(FragmentActivity.FRAGMENTS_TAG);
	          if (p != null) {
	              if (mChildFragmentManager == null) {
	                  instantiateChildFragmentManager();
	              }
	              mChildFragmentManager.restoreAllState(p, mChildNonConfig);
	              mChildNonConfig = null;
	              mChildFragmentManager.dispatchCreate();
	          }
	      }
	  }
	  ```
- ## 3、onCreateView(LayoutInflflater, ViewGroup, Bundle)
  collapsed:: true
	- 在 onCreate() 后就会执行 onCreatView() ，这个方法返回一个 View，默认返回为 null。
	- 当我们需要在 Fragment 中显示布局时，需要重写这个方法，返回要显示的布局。
	- 后面布局销毁时就会调用 onDestroyView() 。
	- ```java
	  @Nullable
	  public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container,
	  										@Nullable Bundle savedInstanceState) {
	  	return null;
	  }
	  ```
	- ## 3.1. onViewCreated
	  collapsed:: true
		- ```java
		  public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
		  }
		  ```
		- onViewCreate() 不是生命周期中的方法，但是却很有用。
		- 它会在 onCreateView() 返回后立即执行，参数中的 view 就是之前创建的 View，因此我们可以在
		- onViewCreate() 中进行布局的初始化，比如这样：
			- ```java
			  @Override
			  public void onViewCreated(final View view, @Nullable final Bundle savedInstanceState) {
			      if (view == null) {
			          return;
			      }
			      mTextView = (TextView) view.findViewById(R.id.tv_content);
			      mBtnSwitchChild = (Button) view.findViewById(R.id.btn_switch_child);
			      Bundle arguments = getArguments();
			      if (arguments != null && mTextView != null &&
			                              !TextUtils.isEmpty(arguments.getString(KEY_TITLE))) {
			          mTextView.setText(arguments.getString(KEY_TITLE));
			      }
			      mBtnSwitchChild.setOnClickListener(new View.OnClickListener() {
			          @Override
			          public void onClick(final View v) {
			          //...
			          });
			      }
			  }
			  ```
- ## 4、**onActivityCreated(Bundle)**
  collapsed:: true
	- onActivityCreated() 会在 Fragment 关联的 Activity 创建好、Fragment 的布局结构初始化完成后调用。
	- 可以在这个方法里做些和布局、状态恢复有关的操作。
	- ```java
	  @CallSuper
	  public void onActivityCreated(@Nullable Bundle savedInstanceState) {
	  	mCalled = true;
	  }
	  ```
	- ## **4.1 onViewStateRestored(Bundle)**
		- ```java
		  @CallSuper
		  public void onViewStateRestored(@Nullable Bundle savedInstanceState) {
		  mCalled = true;
		  }
		  ```
		- onViewStateRestored() 方法会在 onActivityCreated() 结束后调用，用于一个 Fragment 在从
		- 旧的状态恢复时，获取状态 saveInstanceState 恢复状态，比如恢复一个 check box 的状态。
		- 经过这四步，Fragment 创建完成，同步于 Activity 的创建过程。
- ## 5、**onStart()**
  collapsed:: true
	- onStart() 当 Fragment 可见时调用，同步于 Activity 的 onStart() 。
	- ```java
	  @CallSuper
	  public void onStart() {
	      mCalled = true;
	      if (!mLoadersStarted) {
	          mLoadersStarted = true;
	          if (!mCheckedForLoaderManager) {
	              mCheckedForLoaderManager = true;
	              mLoaderManager = mHost.getLoaderManager(mWho, mLoadersStarted,false);
	          }
	          if (mLoaderManager != null) {
	              mLoaderManager.doStart();
	          }
	      }
	  }
	  ```
- ## 6、**onResume()**
  collapsed:: true
	- onResume() 当 Fragment 可见并且可以与用户交互时调用。
	- 它和 Activity 的 onResume() 同步。
	- ```java
	  @CallSuper
	  public void onResume() {
	  	mCalled = true;
	  }
	  ```
- ## 7、**onPause()**
  collapsed:: true
	- onPause() 当 Fragment 不再可见时调用。
	- 也和 Activity的 onPause() 同步
	- ```java
	  @CallSuper
	  public void onPause() {
	  	mCalled = true;
	  }
	  ```
- ## 8、onStop()
  collapsed:: true
	- onStop() 当 Fragment 不再启动时调用，和 Activity.onStop() 一致。
	- ```java
	  @CallSuper
	  public void onStop() {
	  mCalled = true;
	  }
	  ```
- ## 9、onDestroyView()
  collapsed:: true
	- 当 onCreateView() 返回的布局（不论是不是 null）从 Fragment 中解除绑定时调用onDestroyView() 。
	- 下次 Fragment 展示时，会重新创建布局。
	- ```java
	  @CallSuper
	  public void onDestroyView() {
	  mCalled = true;
	  }
	  ```
- ## 10、onDestroy()
  collapsed:: true
	- 当 Fragment 不再使用时会调用 onDestroy() ，它是一个 Fragment 生命周期的倒数第二步。
	- 可以看到这里，调用了 mLoaderManager.doDestroy() ，后面介绍它。
	- ```java
	  @CallSuper
	  public void onDestroy() {
	      mCalled = true;
	      //Log.v("foo", "onDestroy: mCheckedForLoaderManager=" +
	      mCheckedForLoaderManager
	      // + " mLoaderManager=" + mLoaderManager);
	      if (!mCheckedForLoaderManager) {
	          mCheckedForLoaderManager = true;
	          mLoaderManager = mHost.getLoaderManager(mWho, mLoadersStarted,false);
	      }
	      if (mLoaderManager != null) {
	          mLoaderManager.doDestroy();
	      }
	  }
	  ```
- ## 11、**onDetach()**
	- Fragment 生命周期的最后一个方法，当 Fragment 不再和一个 Activity 绑定时调用。
	- Fragment 的 onDestroyView() , onDestroy() , onDetach() 三个对应 Activity 的 onDestroyed()方法。
	- ```java
	  @CallSuper
	  public void onDetach() {
	  	mCalled = true;
	  }
	  ```