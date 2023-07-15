- # 一、职责
	- FragmentManager 定义的任务是由 FragmentManagerImpl 实现的
	- 1、dispatch生命周期的最终调到这里[[#red]]==**dispatchStateChange**==
	- 2、存放事务操作集合
- # 二、获取方式
	- 1、FragmentActivity: FragmentController.getSupportFragmentManager
		- FragmentHostCallback.getFragmentManagerImpl
- # 二、代码
	- ```java
	  final class FragmentManagerImpl extends FragmentManager implements LayoutInflaterFactory {
	          // 存放事务操作的集合 
	          ArrayList<OpGenerator> mPendingActions;
	          Runnable[] mTmpActions;
	          boolean mExecutingActions;
	          ArrayList<Fragment> mActive;
	    
	          // 存入加入的Fragment
	          ArrayList<Fragment> mAdded;
	          ArrayList<Integer> mAvailIndices;
	          ArrayList<BackStackRecord> mBackStack;
	          ArrayList<Fragment> mCreatedMenus;
	          // Must be accessed while locked.
	          ArrayList<BackStackRecord> mBackStackIndices;
	          ArrayList<Integer> mAvailBackStackIndices;
	          ArrayList<OnBackStackChangedListener> mBackStackChangeListeners;
	          private CopyOnWriteArrayList<Pair<FragmentLifecycleCallbacks, Boolean>>
	          mLifecycleCallbacks;
	      	//...
	    		int mCurState = Fragment.INITIALIZING;
	          FragmentHostCallback mHost;
	          FragmentContainer mContainer;
	          Fragment mParent;
	          static Field sAnimationListenerField = null;
	          boolean mNeedMenuInvalidate;
	          boolean mStateSaved;
	          boolean mDestroyed;
	          String mNoTransactionsBecause;
	          boolean mHavePendingDeferredStart;
	  }
	  ```
- # 三、成员
	-