## View.java
collapsed:: true
	- ```java
	  public void requestLayout() {
	          if (mMeasureCache != null) mMeasureCache.clear();
	   
	          if (mAttachInfo != null && mAttachInfo.mViewRequestingLayout == null) {
	              // Only trigger request-during-layout logic if this is the view requesting it,
	              // not the views in its parent hierarchy
	              ViewRootImpl viewRoot = getViewRootImpl();
	              if (viewRoot != null && viewRoot.isInLayout()) {
	                  if (!viewRoot.requestLayoutDuringLayout(this)) {
	                      return;
	                  }
	              }
	              mAttachInfo.mViewRequestingLayout = this;
	          }
	   
	          mPrivateFlags |= PFLAG_FORCE_LAYOUT;
	          mPrivateFlags |= PFLAG_INVALIDATED;
	   
	          if (mParent != null && !mParent.isLayoutRequested()) {
	              mParent.requestLayout();
	          }
	          if (mAttachInfo != null && mAttachInfo.mViewRequestingLayout == this) {
	              mAttachInfo.mViewRequestingLayout = null;
	          }
	      }
	  ```
- 两个步骤，第一个是判断当前是否正在requestLayout，如果是则调用viewRoot.requestLayoutDuringLayout。
  如果没有那么一步一步往上的去调用parent的requestLayout。那么你应该知道最顶端的parent就是ViewRootImpl。我们就直接去看它的requestLayout方法：
- ## ViewRootImpl的requestLayout
  collapsed:: true
	- ```java
	      @Override
	      public void requestLayout() {
	          //如果没有正在处理requestLayout的请求
	          if (!mHandlingLayoutInLayoutRequest) {
	              //线程检验
	              checkThread();
	              mLayoutRequested = true;
	              scheduleTraversals();
	          }
	      }
	      void scheduleTraversals() {
	          if (!mTraversalScheduled) {
	              mTraversalScheduled = true;
	              //暂停了handler的后续消息处理，防止界面刷新的时候出现同步问题
	              mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
	             //将runnable发送给handler执行
	              mChoreographer.postCallback(
	                      Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
	              if (!mUnbufferedInputDispatch) {
	                  scheduleConsumeBatchedInput();
	              }
	              notifyRendererOfFramePending();
	              pokeDrawLockIfNeeded();
	          }
	      }
	  ```
- 其实看到这边基本上已经明了了，接下去我们估计也能猜得到到底是怎么回事儿了。\
- ## mTraversalRunnable
	- ```java
	      final TraversalRunnable mTraversalRunnable = new TraversalRunnable();
	   
	      final class TraversalRunnable implements Runnable {
	          @Override
	          public void run() {
	              doTraversal();
	          }
	      }
	   
	      void doTraversal() {
	          if (mTraversalScheduled) {
	              mTraversalScheduled = false;
	              mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);
	   
	              if (mProfile) {
	                  Debug.startMethodTracing("ViewAncestor");
	              }
	   
	              performTraversals();
	   
	              if (mProfile) {
	                  Debug.stopMethodTracing();
	                  mProfile = false;
	              }
	          }
	      }
	  ```
- 最终requestLayout是这样促成从最根部的ViewRootImpl开始进行measure，layout，draw往下分发。