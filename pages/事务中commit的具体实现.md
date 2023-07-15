- # 一、commit入口代码
  collapsed:: true
	- ```java
	  @Override
	  public int commit() {
	  	return commitInternal(false);
	  }
	  @Override
	  public int commitAllowingStateLoss() {
	  	return commitInternal(true);
	  }
	  
	  int commitInternal(boolean allowStateLoss) {
	      if (mCommitted) throw new IllegalStateException("commit already called");
	      //...
	      mCommitted = true;
	      // 回退栈操作 
	      if (mAddToBackStack) {
	          mIndex = mManager.allocBackStackIndex(this); //更新 index 信息
	      } else {
	          mIndex = -1;
	      }
	      // 把当前类 BackStackRecord 加入到队列中
	      mManager.enqueueAction(this, allowStateLoss); //异步任务入队
	      return mIndex;
	  }
	  
	  // 放入一存放事务的集合  mPendingActions
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
	          scheduleCommit(); //发送任务
	      }
	  }
	  
	  private void scheduleCommit() {
	      synchronized (this) {
	          boolean postponeReady =
	              mPostponedTransactions != null && !mPostponedTransactions.isEmpty();
	          boolean pendingReady = mPendingActions != null && mPendingActions.size() == 1;
	          if (postponeReady || pendingReady) {
	              mHost.getHandler().removeCallbacks(mExecCommit);
	              mHost.getHandler().post(mExecCommit);
	          }
	      }
	  }
	  ```
	- FragmentManager.enqueueAction() 最终是使用 Handler 实现的异步执行。
- # 二、看Handler 具体执行的任务
  collapsed:: true
	- 答案就是 Handler 发送的任务 mExecCommit :
	  collapsed:: true
		- ```java
		  Runnable mExecCommit = new Runnable() {
		      @Override
		      public void run() {
		          execPendingActions();
		      }
		  };
		  
		  /**
		  * Only call from main thread!
		  * 更新 UI 嘛，肯定得在主线程
		  */
		  public boolean execPendingActions() {
		      // 确认是否允许丢失状态
		      ensureExecReady(true);
		      boolean didSomething = false;
		      while (generateOpsForPendingActions(mTmpRecords, mTmpIsPop)) {
		          mExecutingActions = true;
		          try {
		              optimizeAndExecuteOps(mTmpRecords, mTmpIsPop); //这里是入口
		          } finally {
		              cleanupExec();
		          }
		          didSomething = true;
		      }
		      doPendingDeferredStart();
		      return didSomething;
		  }
		  
		  private void optimizeAndExecuteOps(ArrayList<BackStackRecord> records,
		  												ArrayList<Boolean> isRecordPop) {
		      if (records == null || records.isEmpty()) {
		          return;
		      }
		      if (isRecordPop == null || records.size() != isRecordPop.size()) {
		          throw new IllegalStateException("Internal error with the back stack records");
		      }
		      // Force start of any postponed transactions that interact with scheduled transactions:
		      executePostponedTransaction(records, isRecordPop);
		      final int numRecords = records.size();
		      int startIndex = 0;
		      for (int recordNum = 0; recordNum < numRecords; recordNum++) {
		          final boolean canOptimize = records.get(recordNum).mAllowOptimization;
		          if (!canOptimize) {
		              // execute all previous transactions
		              if (startIndex != recordNum) {
		                //这里将 Ops 过滤一遍
		                executeOpsTogether(records, isRecordPop, startIndex, recordNum);
		              }
		              // execute all unoptimized pop operations together or one add operation
		              //...
		          }
		          if (startIndex != numRecords) {
		              executeOpsTogether(records, isRecordPop, startIndex, numRecords);
		          }
		      }
		  }
		    
		    
		  private void executeOpsTogether(ArrayList<BackStackRecord> records,
		      			ArrayList<Boolean> isRecordPop, int startIndex, int endIndex) {
		      final boolean allowOptimization =
		      records.get(startIndex).mAllowOptimization;
		      boolean addToBackStack = false;
		      if (mTmpAddedFragments == null) {
		          mTmpAddedFragments = new ArrayList<>();
		      } else {
		          mTmpAddedFragments.clear();
		      }
		      if (mAdded != null) {
		          mTmpAddedFragments.addAll(mAdded);
		      }
		      for (int recordNum = startIndex; recordNum < endIndex; recordNum++) {
		          final BackStackRecord record = records.get(recordNum);
		          final boolean isPop = isRecordPop.get(recordNum);
		          if (!isPop) {
		              record.expandReplaceOps(mTmpAddedFragments); //修改状态
		          } else {
		              record.trackAddedFragmentsInPop(mTmpAddedFragments);
		          }
		          addToBackStack = addToBackStack || record.mAddToBackStack;
		      }
		      mTmpAddedFragments.clear();
		      if (!allowOptimization) {
		          FragmentTransition.startTransitions(this, records, isRecordPop,
		          startIndex, endIndex,false);
		      }
		      //真正处理的入口
		      executeOps(records, isRecordPop, startIndex, endIndex);
		      int postponeIndex = endIndex;
		      if (allowOptimization) {
		          ArraySet<Fragment> addedFragments = new ArraySet<>();
		          addAddedFragments(addedFragments);
		          postponeIndex = postponePostponableTransactions(records,
		                                      isRecordPop, startIndex, endIndex, addedFragments);
		            //名字就能看出来作用
		          makeRemovedFragmentsInvisible(addedFragments); 
		      }
		  
		      if (postponeIndex != startIndex && allowOptimization) {
		          // need to run something now
		          FragmentTransition.startTransitions(this, records, isRecordPop,
		                              startIndex, postponeIndex, true);
		          moveToState(mCurState, true);
		      }
		  //...
		  }
		  
		  //修改 Ops 状态，这一步还没有真正处理状态
		  expandReplaceOps(ArrayList<Fragment> added) {
		    
		      for (int opNum = 0; opNum < mOps.size(); opNum++) {
		          final Op op = mOps.get(opNum);
		          switch (op.cmd) {
		              case OP_ADD:
		              case OP_ATTACH:
		                  added.add(op.fragment);
		              break;
		              case OP_REMOVE:
		              case OP_DETACH:
		                  added.remove(op.fragment);
		              break;
		              case OP_REPLACE: {
		                    Fragment f = op.fragment;
		                    int containerId = f.mContainerId;
		                    boolean alreadyAdded = false;
		                    for (int i = added.size() - 1; i >= 0; i--) {
		                        Fragment old = added.get(i);
		                        if (old.mContainerId == containerId) {
		                            if (old == f) {
		                                alreadyAdded = true;
		                            } else {
		                                Op removeOp = new Op();
		                                   //可以看到，替换也是通过删除实现的
		                                removeOp.cmd = OP_REMOVE;
		                                removeOp.fragment = old;
		                                removeOp.enterAnim = op.enterAnim;
		                                removeOp.popEnterAnim = op.popEnterAnim;
		                                removeOp.exitAnim = op.exitAnim;
		                                removeOp.popExitAnim = op.popExitAnim;
		                                mOps.add(opNum, removeOp);
		                                added.remove(old);
		                                opNum++;
		                            }
		                        }
		                    }
		                    if (alreadyAdded) {
		                        mOps.remove(opNum);
		                        opNum--;
		                    } else {
		                        op.cmd = OP_ADD;
		                        added.add(f);
		                    }
		              }
		              break;
		          }
		       }
		  }
		  
		  //设置将要被移除的 Fragment 为不可见的最终实现
		  private void makeRemovedFragmentsInvisible(ArraySet<Fragment> fragments) {
		      final int numAdded = fragments.size();
		      for (int i = 0; i < numAdded; i++) {
		          final Fragment fragment = fragments.valueAt(i);
		          if (!fragment.mAdded) {
		                //获取 Fragment 的布局，设置状态
		              final View view = fragment.getView(); 
		              if (Build.VERSION.SDK_INT < Build.VERSION_CODES.HONEYCOMB) {
		                  fragment.getView().setVisibility(View.INVISIBLE);
		              } else { //高版本设置透明度
		                  fragment.mPostponedAlpha = view.getAlpha();
		                  view.setAlpha(0f);
		              }
		          }
		      }
		  }
		  ```
	- 代码多了一点，但我们终于找到了最终的实现：Handler 异步发到主线，调度执行后，聚合、修改 Ops 的状态，然后遍历、修改 Fragment 栈中的 View 的状态。
- # 三、真正根据 Ops 状态进行操作
	- 前面主要是对 Fragment 的包装类 Ops 进行一些状态修改，真正根据 Ops 状态进行操作在这个部分
		- ```java
		  /**
		  * Executes the operations contained within this transaction. The Fragment
		  states will only
		  * be modified if optimizations are not allowed.
		  */
		  void executeOps() {
		      final int numOps = mOps.size();
		      for (int opNum = 0; opNum < numOps; opNum++) {
		          final Op op = mOps.get(opNum);
		          final Fragment f = op.fragment;
		          f.setNextTransition(mTransition, mTransitionStyle);
		          switch (op.cmd) {
		              case OP_ADD:
		                  f.setNextAnim(op.enterAnim);
		                  mManager.addFragment(f, false);
		              break;
		              case OP_REMOVE:
		                  f.setNextAnim(op.exitAnim);
		                  mManager.removeFragment(f);
		              break;
		              case OP_HIDE:
		                  f.setNextAnim(op.exitAnim);
		                  mManager.hideFragment(f);
		              break;
		              case OP_SHOW:
		                  f.setNextAnim(op.enterAnim);
		                  mManager.showFragment(f);
		              break;
		              case OP_DETACH:
		                  f.setNextAnim(op.exitAnim);
		                  mManager.detachFragment(f);
		              break;
		              case OP_ATTACH:
		                  f.setNextAnim(op.enterAnim);
		                  mManager.attachFragment(f);
		              break;
		              default:
		                  throw new IllegalArgumentException("Unknown cmd: " + op.cmd);
		          }
		          if (!mAllowOptimization && op.cmd != OP_ADD) {
		              mManager.moveFragmentToExpectedState(f);
		          }
		      }
		      if (!mAllowOptimization) {
		          // Added fragments are added at the end to comply with prior
		          behavior.mManager.moveToState(mManager.mCurState, true);
		      }
		  }
		  ```
- # 四、[[commit最终方法moveToState修改状态值]]