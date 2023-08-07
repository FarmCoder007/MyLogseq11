# 源码-四级缓存
	- ```java
	  ViewHolder tryGetViewHolderForPositionByDeadline(int position,
	                  boolean dryRun, long deadlineNs) {
	              if (position < 0 || position >= mState.getItemCount()) {
	                  throw new IndexOutOfBoundsException("Invalid item position " + position
	                          + "(" + position + "). Item count:" + mState.getItemCount()
	                          + exceptionLabel());
	              }
	              boolean fromScrapOrHiddenOrCache = false;
	              ViewHolder holder = null;
	              // 0) If there is a changed scrap, try to find from there
	              if (mState.isPreLayout()) {
	                  holder = getChangedScrapViewForPosition(position);
	                  fromScrapOrHiddenOrCache = holder != null;
	              }
	              // 1) Find by position from scrap/hidden list/cache
	              if (holder == null) {
	                  holder = getScrapOrHiddenOrCachedHolderForPosition(position, dryRun);
	                  if (holder != null) {
	                      if (!validateViewHolderForOffsetPosition(holder)) {
	                          // recycle holder (and unscrap if relevant) since it can't be used
	                          if (!dryRun) {
	                              // we would like to recycle this but need to make sure it is not used by
	                              // animation logic etc.
	                              holder.addFlags(ViewHolder.FLAG_INVALID);
	                              if (holder.isScrap()) {
	                                  removeDetachedView(holder.itemView, false);
	                                  holder.unScrap();
	                              } else if (holder.wasReturnedFromScrap()) {
	                                  holder.clearReturnedFromScrapFlag();
	                              }
	                              recycleViewHolderInternal(holder);
	                          }
	                          holder = null;
	                      } else {
	                          fromScrapOrHiddenOrCache = true;
	                      }
	                  }
	              }
	              if (holder == null) {
	                  final int offsetPosition = mAdapterHelper.findPositionOffset(position);
	                  if (offsetPosition < 0 || offsetPosition >= mAdapter.getItemCount()) {
	                      throw new IndexOutOfBoundsException("Inconsistency detected. Invalid item "
	                              + "position " + position + "(offset:" + offsetPosition + ")."
	                              + "state:" + mState.getItemCount() + exceptionLabel());
	                  }
	  
	                  final int type = mAdapter.getItemViewType(offsetPosition);
	                  // 2) Find from scrap/cache via stable ids, if exists
	                  if (mAdapter.hasStableIds()) {
	                      holder = getScrapOrCachedViewForId(mAdapter.getItemId(offsetPosition),
	                              type, dryRun);
	                      if (holder != null) {
	                          // update position
	                          holder.mPosition = offsetPosition;
	                          fromScrapOrHiddenOrCache = true;
	                      }
	                  }
	                  if (holder == null && mViewCacheExtension != null) {
	                      // We are NOT sending the offsetPosition because LayoutManager does not
	                      // know it.
	                      final View view = mViewCacheExtension
	                              .getViewForPositionAndType(this, position, type);
	                      if (view != null) {
	                          holder = getChildViewHolder(view);
	                          if (holder == null) {
	                              throw new IllegalArgumentException("getViewForPositionAndType returned"
	                                      + " a view which does not have a ViewHolder"
	                                      + exceptionLabel());
	                          } else if (holder.shouldIgnore()) {
	                              throw new IllegalArgumentException("getViewForPositionAndType returned"
	                                      + " a view that is ignored. You must call stopIgnoring before"
	                                      + " returning this view." + exceptionLabel());
	                          }
	                      }
	                  }
	                  if (holder == null) { // fallback to pool
	                      if (DEBUG) {
	                          Log.d(TAG, "tryGetViewHolderForPositionByDeadline("
	                                  + position + ") fetching from shared pool");
	                      }
	                      holder = getRecycledViewPool().getRecycledView(type);
	                      if (holder != null) {
	                          holder.resetInternal();
	                          if (FORCE_INVALIDATE_DISPLAY_LIST) {
	                              invalidateDisplayListInt(holder);
	                          }
	                      }
	                  }
	                  if (holder == null) {
	                      long start = getNanoTime();
	                      if (deadlineNs != FOREVER_NS
	                              && !mRecyclerPool.willCreateInTime(type, start, deadlineNs)) {
	                          // abort - we have a deadline we can't meet
	                          return null;
	                      }
	                      holder = mAdapter.createViewHolder(RecyclerView.this, type);
	                      if (ALLOW_THREAD_GAP_WORK) {
	                          // only bother finding nested RV if prefetching
	                          RecyclerView innerView = findNestedRecyclerView(holder.itemView);
	                          if (innerView != null) {
	                              holder.mNestedRecyclerView = new WeakReference<>(innerView);
	                          }
	                      }
	  
	                      long end = getNanoTime();
	                      mRecyclerPool.factorInCreateTime(type, end - start);
	                      if (DEBUG) {
	                          Log.d(TAG, "tryGetViewHolderForPositionByDeadline created new ViewHolder");
	                      }
	                  }
	              }
	  
	              // This is very ugly but the only place we can grab this information
	              // before the View is rebound and returned to the LayoutManager for post layout ops.
	              // We don't need this in pre-layout since the VH is not updated by the LM.
	              if (fromScrapOrHiddenOrCache && !mState.isPreLayout() && holder
	                      .hasAnyOfTheFlags(ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST)) {
	                  holder.setFlags(0, ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST);
	                  if (mState.mRunSimpleAnimations) {
	                      int changeFlags = ItemAnimator
	                              .buildAdapterChangeFlagsForAnimations(holder);
	                      changeFlags |= ItemAnimator.FLAG_APPEARED_IN_PRE_LAYOUT;
	                      final ItemHolderInfo info = mItemAnimator.recordPreLayoutInformation(mState,
	                              holder, changeFlags, holder.getUnmodifiedPayloads());
	                      recordAnimationInfoIfBouncedHiddenView(holder, info);
	                  }
	              }
	  
	              boolean bound = false;
	              if (mState.isPreLayout() && holder.isBound()) {
	                  // do not update unless we absolutely have to.
	                  holder.mPreLayoutPosition = position;
	              } else if (!holder.isBound() || holder.needsUpdate() || holder.isInvalid()) {
	                  if (DEBUG && holder.isRemoved()) {
	                      throw new IllegalStateException("Removed holder should be bound and it should"
	                              + " come here only in pre-layout. Holder: " + holder
	                              + exceptionLabel());
	                  }
	                  final int offsetPosition = mAdapterHelper.findPositionOffset(position);
	                  bound = tryBindViewHolderByDeadline(holder, offsetPosition, position, deadlineNs);
	              }
	  
	              final ViewGroup.LayoutParams lp = holder.itemView.getLayoutParams();
	              final LayoutParams rvLayoutParams;
	              if (lp == null) {
	                  rvLayoutParams = (LayoutParams) generateDefaultLayoutParams();
	                  holder.itemView.setLayoutParams(rvLayoutParams);
	              } else if (!checkLayoutParams(lp)) {
	                  rvLayoutParams = (LayoutParams) generateLayoutParams(lp);
	                  holder.itemView.setLayoutParams(rvLayoutParams);
	              } else {
	                  rvLayoutParams = (LayoutParams) lp;
	              }
	              rvLayoutParams.mViewHolder = holder;
	              rvLayoutParams.mPendingInvalidate = fromScrapOrHiddenOrCache && bound;
	              return holder;
	          }
	  
	  ```
- # [[四级缓存介绍]]，怎么从集合里取缓存
	- ## [[一级缓存:用来缓存屏幕内的viewHolder]]
	- ## 二级缓存：取移除屏幕外的ViewHolder
	  collapsed:: true
		- ## 方法：getScrapOrHiddenOrCachedHolderForPosition
		  collapsed:: true
			- ```java
			  ViewHolder getScrapOrHiddenOrCachedHolderForPosition(int position, boolean dryRun) {
			              final int scrapCount = mAttachedScrap.size();
			  
			              // Try first for an exact, non-invalid match from scrap.
			              for (int i = 0; i < scrapCount; i++) {
			                  final ViewHolder holder = mAttachedScrap.get(i);
			                  if (!holder.wasReturnedFromScrap() && holder.getLayoutPosition() == position
			                          && !holder.isInvalid() && (mState.mInPreLayout || !holder.isRemoved())) {
			                      holder.addFlags(ViewHolder.FLAG_RETURNED_FROM_SCRAP);
			                      return holder;
			                  }
			              }
			  
			              if (!dryRun) {
			                  View view = mChildHelper.findHiddenNonRemovedView(position);
			                  if (view != null) {
			                      // This View is good to be used. We just need to unhide, detach and move to the
			                      // scrap list.
			                      final ViewHolder vh = getChildViewHolderInt(view);
			                      mChildHelper.unhide(view);
			                      int layoutIndex = mChildHelper.indexOfChild(view);
			                      if (layoutIndex == RecyclerView.NO_POSITION) {
			                          throw new IllegalStateException("layout index should not be -1 after "
			                                  + "unhiding a view:" + vh + exceptionLabel());
			                      }
			                      mChildHelper.detachViewFromParent(layoutIndex);
			                      scrapView(view);
			                      vh.addFlags(ViewHolder.FLAG_RETURNED_FROM_SCRAP
			                              | ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST);
			                      return vh;
			                  }
			              }
			  
			              // Search in our first-level recycled view cache.
			              final int cacheSize = mCachedViews.size();
			              for (int i = 0; i < cacheSize; i++) {
			                  final ViewHolder holder = mCachedViews.get(i);
			                  // invalid view holders may be in cache if adapter has stable ids as they can be
			                  // retrieved via getScrapOrCachedViewForId
			                  if (!holder.isInvalid() && holder.getLayoutPosition() == position
			                          && !holder.isAttachedToTransitionOverlay()) {
			                      if (!dryRun) {
			                          mCachedViews.remove(i);
			                      }
			                      if (DEBUG) {
			                          Log.d(TAG, "getScrapOrHiddenOrCachedHolderForPosition(" + position
			                                  + ") found match in cache: " + holder);
			                      }
			                      return holder;
			                  }
			              }
			              return null;
			          }
			  
			  ```
		- 通过position   从 mAttachedScrap  和 mCachedViews获取
		- ## 方法：getScrapOrCachedViewForId
		  collapsed:: true
			- ```java
			  ViewHolder getScrapOrCachedViewForId(long id, int type, boolean dryRun) {
			              // Look in our attached views first
			              final int count = mAttachedScrap.size();
			              for (int i = count - 1; i >= 0; i--) {
			                  final ViewHolder holder = mAttachedScrap.get(i);
			                  if (holder.getItemId() == id && !holder.wasReturnedFromScrap()) {
			                      if (type == holder.getItemViewType()) {
			                          holder.addFlags(ViewHolder.FLAG_RETURNED_FROM_SCRAP);
			                          if (holder.isRemoved()) {
			                              // this might be valid in two cases:
			                              // > item is removed but we are in pre-layout pass
			                              // >> do nothing. return as is. make sure we don't rebind
			                              // > item is removed then added to another position and we are in
			                              // post layout.
			                              // >> remove removed and invalid flags, add update flag to rebind
			                              // because item was invisible to us and we don't know what happened in
			                              // between.
			                              if (!mState.isPreLayout()) {
			                                  holder.setFlags(ViewHolder.FLAG_UPDATE, ViewHolder.FLAG_UPDATE
			                                          | ViewHolder.FLAG_INVALID | ViewHolder.FLAG_REMOVED);
			                              }
			                          }
			                          return holder;
			                      } else if (!dryRun) {
			                          // if we are running animations, it is actually better to keep it in scrap
			                          // but this would force layout manager to lay it out which would be bad.
			                          // Recycle this scrap. Type mismatch.
			                          mAttachedScrap.remove(i);
			                          removeDetachedView(holder.itemView, false);
			                          quickRecycleScrapView(holder.itemView);
			                      }
			                  }
			              }
			  
			              // Search the first-level cache
			              final int cacheSize = mCachedViews.size();
			              for (int i = cacheSize - 1; i >= 0; i--) {
			                  final ViewHolder holder = mCachedViews.get(i);
			                  if (holder.getItemId() == id && !holder.isAttachedToTransitionOverlay()) {
			                      if (type == holder.getItemViewType()) {
			                          if (!dryRun) {
			                              mCachedViews.remove(i);
			                          }
			                          return holder;
			                      } else if (!dryRun) {
			                          recycleCachedViewAt(i);
			                          return null;
			                      }
			                  }
			              }
			              return null;
			          }
			  ```
		- 通过viewType和 id 从 mAttachedScrap  和 mCachedViews获取
	- ## 三级缓存：取自定义扩展缓存的viewHodler
	  collapsed:: true
		- ## 方法：
			- ```java
			   final View view = mViewCacheExtension
			      .getViewForPositionAndType(this, position, type);
			  ```
		- ## getViewForPositionAndType抽象的自己去重写
		- 自定义缓存-局部刷新的时候用的到
	- ## 四级缓存：从缓冲池里获取getRecycledViewPool().getRecycledView(type);
		- getRecycledViewPool
			- ```java
			          RecycledViewPool getRecycledViewPool() {
			              if (mRecyclerPool == null) {
			                  mRecyclerPool = new RecycledViewPool();
			              }
			              return mRecyclerPool;
			          }
			  ```
		- getRecycledView
			- ```java
			  
			  
			          public ViewHolder getRecycledView(int viewType) {
			              final ScrapData scrapData = mScrap.get(viewType);
			              if (scrapData != null && !scrapData.mScrapHeap.isEmpty()) {
			                  final ArrayList<ViewHolder> scrapHeap = scrapData.mScrapHeap;
			                  for (int i = scrapHeap.size() - 1; i >= 0; i--) {
			                      if (!scrapHeap.get(i).isAttachedToTransitionOverlay()) {
			                          return scrapHeap.remove(i);
			                      }
			                  }
			              }
			              return null;
			          }
			  ```
	- ## createViewHolder-没有缓存就得重新创建了.最终调用到RD重写的onCreateViewHolder
	  collapsed:: true
		- ```java
		          public final VH createViewHolder(@NonNull ViewGroup parent, int viewType) {
		              try {
		                  TraceCompat.beginSection(TRACE_CREATE_VIEW_TAG);
		                  final VH holder = onCreateViewHolder(parent, viewType);
		                  if (holder.itemView.getParent() != null) {
		                      throw new IllegalStateException("ViewHolder views must not be attached when"
		                              + " created. Ensure that you are not passing 'true' to the attachToRoot"
		                              + " parameter of LayoutInflater.inflate(..., boolean attachToRoot)");
		                  }
		                  holder.mItemViewType = viewType;
		                  return holder;
		              } finally {
		                  TraceCompat.endSection();
		              }
		          }
		  ```
	- ## tryBindViewHolderByDeadline-创建viewHolder后 绑定
	  collapsed:: true
		- 最终调用到RD重写的 onBindViewHolder
		- ```java
		          private boolean tryBindViewHolderByDeadline(@NonNull ViewHolder holder, int offsetPosition,
		                  int position, long deadlineNs) {
		              holder.mOwnerRecyclerView = RecyclerView.this;
		              final int viewType = holder.getItemViewType();
		              long startBindNs = getNanoTime();
		              if (deadlineNs != FOREVER_NS
		                      && !mRecyclerPool.willBindInTime(viewType, startBindNs, deadlineNs)) {
		                  // abort - we have a deadline we can't meet
		                  return false;
		              }
		              mAdapter.bindViewHolder(holder, offsetPosition);
		              long endBindNs = getNanoTime();
		              mRecyclerPool.factorInBindTime(holder.getItemViewType(), endBindNs - startBindNs);
		              attachAccessibilityDelegateOnBind(holder);
		              if (mState.isPreLayout()) {
		                  holder.mPreLayoutPosition = position;
		              }
		              return true;
		          }
		  
		  ```
		-