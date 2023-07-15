- # 一、定义
	- FragmentManager 是一个抽象类，定义了一些和 Fragment 相关的操作和内部类/接口
	- 它定义了对一个 Activity/Fragment 中 添加进来的**Fragment ****列表**、**Fragment ****回退栈**的操作、管理。
	- ```java
	  public abstract class FragmentManager {...}
	  ```
- # 二、定义的方法
	- 可以看到，定义的方法有很多是异步执行的，后面看看它究竟是如何实现的异步。
	- ```java
	  //开启一系列对 Fragments 的操作
	  public abstract FragmentTransaction beginTransaction();
	  //FragmentTransaction.commit() 是异步执行的，如果你想立即执行，可以调用这个方法
	  public abstract boolean executePendingTransactions();
	  //根据 ID 找到从 XML 解析出来的或者事务中添加的 Fragment
	  //首先会找添加到 FragmentManager 中的，找不到就去回退栈里找
	  public abstract Fragment findFragmentById(@IdRes int id);
	  //跟上面的类似，不同的是使用 tag 进行查找
	  public abstract Fragment findFragmentByTag(String tag);
	  //弹出回退栈中栈顶的 Fragment，异步执行的
	  public abstract void popBackStack();
	  //立即弹出回退栈中栈顶的，直接执行哦
	  public abstract boolean popBackStackImmediate();
	  //返回栈顶符合名称的，如果传入的 name 不为空，在栈中间找到了 Fragment，那将弹出这个
	  // Fragment 上面的所有 Fragment
	  //有点类似启动模式的 singleTask 的感觉
	  //如果传入的 name 为 null，那就和 popBackStack() 一样了
	  //异步执行
	  public abstract void popBackStack(String name, int flags);
	  //同步版的上面
	  public abstract boolean popBackStackImmediate(String name, int flags);
	  //和使用 name 查找、弹出一样
	  //不同的是这里的 id 是 FragmentTransaction.commit() 返回的 id
	  public abstract void popBackStack(int id, int flags);
	  //你懂得
	  public abstract boolean popBackStackImmediate(int id, int flags);
	  //获取回退栈中的元素个数
	  public abstract int getBackStackEntryCount();
	  //根据索引获取回退栈中的某个元素
	  public abstract BackStackEntry getBackStackEntryAt(int index);
	  //添加或者移除一个监听器
	  public abstract void
	  addOnBackStackChangedListener(OnBackStackChangedListener listener);
	  public abstract void
	  removeOnBackStackChangedListener(OnBackStackChangedListener listener);
	  //还定义了将一个 Fragment 实例作为参数传递
	  public abstract void putFragment(Bundle bundle, String key, Fragment fragment);
	  public abstract Fragment getFragment(Bundle bundle, String key);
	  //获取 manager 中所有添加进来的 Fragment
	  public abstract List<Fragment> getFragments();
	  ```
- # 三、内部类/接口
	- ## [[BackStackEntry]]：Fragment 后退栈中的一个元素
	- ## [[onBackStackChangedListener]]：后退栈变动监听器
	- ## FragmentLifecycleCallbacks: FragmentManager 中的 Fragment 生命周期监听
		-
	-
- # 四、实现类 FragmentManagerImpl
	- ## [[FragmentManagerImpl]]
	- ## [[FragmentContainer]]
	- ## [[FragmentHostCallback]]
		- 我们再看看他对 FragmentManager 定义的关键方法是如何实现的。
			- ```java
			  @Override
			  public FragmentTransaction beginTransaction() {
			  	return new BackStackRecord(this);
			  }
			  ```
		- beginTransaction() 返回一个新的 BackStackRecord ，我们后面介绍。前面提到了， popBackStack() 是一个异步操作，它是如何实现异步的呢？
			- ```java
			  @Override
			  public void popBackStack() {
			  	enqueueAction(new PopBackStackState(null, -1, 0), false);
			  }
			  public void enqueueAction(OpGenerator action, boolean allowStateLoss) {
			      if (!allowStateLoss) {
			      	checkStateLoss();
			      }
			      synchronized (this) {
			          if (mDestroyed || mHost == null) {
			          	throw new IllegalStateException("Activity has been destroyed");
			          }
			          if (mPendingActions == null) {
			          	mPendingActions = new ArrayList<>();
			          }
			          mPendingActions.add(action);
			          scheduleCommit();
			      }
			  }
			  private void scheduleCommit() {
			      synchronized (this) {
			          boolean postponeReady =
			          mPostponedTransactions != null &&
			          !mPostponedTransactions.isEmpty();
			            boolean pendingReady = mPendingActions != null &&
			          mPendingActions.size() == 1;
			          if (postponeReady || pendingReady) {
			              mHost.getHandler().removeCallbacks(mExecCommit);
			              mHost.getHandler().post(mExecCommit);
			          }
			      }
			  }
			  ```
		- 可以看到，调用到最后，是调用宿主中的 Handler 来发送任务的，so easy 嘛。其他的异步执行也是类似，就不赘述了。
		-
	- 后退栈相关方法：mBackStack
		- ```java
		  ArrayList<BackStackRecord> mBackStack;
		  @Override
		  public int getBackStackEntryCount() {
		  return mBackStack != null ? mBackStack.size() : 0;
		  }
		  @Override
		  public BackStackEntry getBackStackEntryAt(int index) {
		  return mBackStack.get(index);
		  }
		  ```
		- 可以看到，开始事务和后退栈，返回/操作的都是 [[BackStackRecord]] ，我们来了解了解它是何方神圣
-
	-