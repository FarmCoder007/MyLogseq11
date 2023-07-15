- # 一、概述
  collapsed:: true
	- 它提供了 Fragment 需要的信息，也定义了 Fragment 宿主应该做的操作：
	- ```java
	  public abstract class FragmentHostCallback<E> extends FragmentContainer {
	      private final Activity mActivity;
	      final Context mContext;
	      private final Handler mHandler;
	      final int mWindowAnimations;
	      final FragmentManagerImpl mFragmentManager = new FragmentManagerImpl();
	      //...
	  }
	  ```
	-
	- 我们知道，一般来说 Fragment 的宿主就两种：Activity,Fragment
	- 比如 FragmentActivity 的内部类 HostCallbacks 就实现了这个抽象类
		- ```java
		  class HostCallbacks extends FragmentHostCallback<FragmentActivity> {
		      public HostCallbacks() {
		      	super(FragmentActivity.this /*fragmentActivity*/);
		      }
		  //...
		      @Override
		      public LayoutInflater onGetLayoutInflater() {
		          return
		              FragmentActivity.this.getLayoutInflater().cloneInContext(FragmentActivity.t
		              his);
		      }
		      @Override
		      public FragmentActivity onGetHost() {
		      	return FragmentActivity.this;
		      }
		      @Override
		      public void onStartActivityFromFragment(Fragment fragment, Intent
		      													intent, int requestCode) {
		          FragmentActivity.this.startActivityFromFragment(fragment, intent,
		          requestCode);
		      }
		      @Override
		      public void onStartActivityFromFragment(Fragment fragment, Intent intent, 
		                                              int requestCode, @Nullable Bundle options) {
		      	FragmentActivity.this.startActivityFromFragment(fragment, intent,
		      												requestCode, options);
		      }
		      @Override
		      public void onRequestPermissionsFromFragment(@NonNull Fragment
		      					fragment, @NonNull String[] permissions, int requestCode) {
		          FragmentActivity.this.requestPermissionsFromFragment(fragment,
		              permissions, requestCode);
		      }
		      @Override
		      public boolean onShouldShowRequestPermissionRationale(@NonNull String
		      																permission) {
		          return ActivityCompat.shouldShowRequestPermissionRationale(
		          FragmentActivity.this, permission);
		      }
		      
		      @Override
		      public boolean onHasWindowAnimations() {
		      }
		      
		      @Override
		      public void onAttachFragment(Fragment fragment) {
		      		FragmentActivity.this.onAttachFragment(fragment);
		      }
		      @Nullable
		      @Override
		      public View onFindViewById(int id) {
		      	return FragmentActivity.this.findViewById(id);
		      }
		      @Override
		      public boolean onHasView() {
		          final Window w = getWindow();
		          return (w != null && w.peekDecorView() != null);
		      }
		  }
		  ```
- # 二、成员
	- ## [[FragmentManagerImpl]]
		- 1、dispatch生命周期的最终调到这里[[#red]]==**dispatchStateChange**==
		- 2、存放事务操作集合
	- ## [[BackStackRecord]]
		- 各种事务的操作，add/replace