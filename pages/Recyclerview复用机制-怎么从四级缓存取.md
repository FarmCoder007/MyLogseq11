- >先从滑动开始看，肯定需要滑动出去，回收，划进来的复用。
- # recyclerVIew
  collapsed:: true
	- ## 1、onTouchEvent的Action——Move事件
	  collapsed:: true
		- ```java
		      @Override
		      public boolean onTouchEvent(MotionEvent e) {
		        
		        
		          case MotionEvent.ACTION_MOVE: {
		                  final int index = e.findPointerIndex(mScrollPointerId);
		                  if (index < 0) {
		                      Log.e(TAG, "Error processing scroll; pointer index for id "
		                              + mScrollPointerId + " not found. Did any MotionEvents get skipped?");
		                      return false;
		                  }
		  
		                  final int x = (int) (e.getX(index) + 0.5f);
		                  final int y = (int) (e.getY(index) + 0.5f);
		                  int dx = mLastTouchX - x;
		                  int dy = mLastTouchY - y;
		  
		                  if (mScrollState != SCROLL_STATE_DRAGGING) {
		                      boolean startScroll = false;
		                      if (canScrollHorizontally) {
		                          if (dx > 0) {
		                              dx = Math.max(0, dx - mTouchSlop);
		                          } else {
		                              dx = Math.min(0, dx + mTouchSlop);
		                          }
		                          if (dx != 0) {
		                              startScroll = true;
		                          }
		                      }
		                      if (canScrollVertically) {
		                          if (dy > 0) {
		                              dy = Math.max(0, dy - mTouchSlop);
		                          } else {
		                              dy = Math.min(0, dy + mTouchSlop);
		                          }
		                          if (dy != 0) {
		                              startScroll = true;
		                          }
		                      }
		                      if (startScroll) {
		                          setScrollState(SCROLL_STATE_DRAGGING);
		                      }
		                  }
		  
		                  if (mScrollState == SCROLL_STATE_DRAGGING) {
		                      mReusableIntPair[0] = 0;
		                      mReusableIntPair[1] = 0;
		                      if (dispatchNestedPreScroll(
		                              canScrollHorizontally ? dx : 0,
		                              canScrollVertically ? dy : 0,
		                              mReusableIntPair, mScrollOffset, TYPE_TOUCH
		                      )) {
		                          dx -= mReusableIntPair[0];
		                          dy -= mReusableIntPair[1];
		                          // Updated the nested offsets
		                          mNestedOffsets[0] += mScrollOffset[0];
		                          mNestedOffsets[1] += mScrollOffset[1];
		                          // Scroll has initiated, prevent parents from intercepting
		                          getParent().requestDisallowInterceptTouchEvent(true);
		                      }
		  
		                      mLastTouchX = x - mScrollOffset[0];
		                      mLastTouchY = y - mScrollOffset[1];
		  
		                      if (scrollByInternal(
		                              canScrollHorizontally ? dx : 0,
		                              canScrollVertically ? dy : 0,
		                              e)) {
		                          getParent().requestDisallowInterceptTouchEvent(true);
		                      }
		                      if (mGapWorker != null && (dx != 0 || dy != 0)) {
		                          mGapWorker.postFromTraversal(this, dx, dy);
		                      }
		                  }
		              } break;
		  
		        
		        
		        
		        
		        
		      }
		  ```
	- ## 2、scrollByInternal
	  collapsed:: true
		- ```java
		   boolean scrollByInternal(int x, int y, MotionEvent ev) {
		          int unconsumedX = 0;
		          int unconsumedY = 0;
		          int consumedX = 0;
		          int consumedY = 0;
		  
		          consumePendingUpdateOperations();
		          if (mAdapter != null) {
		              mReusableIntPair[0] = 0;
		              mReusableIntPair[1] = 0;
		              scrollStep(x, y, mReusableIntPair);
		              consumedX = mReusableIntPair[0];
		              consumedY = mReusableIntPair[1];
		              unconsumedX = x - consumedX;
		              unconsumedY = y - consumedY;
		          }
		          if (!mItemDecorations.isEmpty()) {
		              invalidate();
		          }
		  
		          mReusableIntPair[0] = 0;
		          mReusableIntPair[1] = 0;
		          dispatchNestedScroll(consumedX, consumedY, unconsumedX, unconsumedY, mScrollOffset,
		                  TYPE_TOUCH, mReusableIntPair);
		          unconsumedX -= mReusableIntPair[0];
		          unconsumedY -= mReusableIntPair[1];
		          boolean consumedNestedScroll = mReusableIntPair[0] != 0 || mReusableIntPair[1] != 0;
		  
		          // Update the last touch co-ords, taking any scroll offset into account
		          mLastTouchX -= mScrollOffset[0];
		          mLastTouchY -= mScrollOffset[1];
		          mNestedOffsets[0] += mScrollOffset[0];
		          mNestedOffsets[1] += mScrollOffset[1];
		  
		          if (getOverScrollMode() != View.OVER_SCROLL_NEVER) {
		              if (ev != null && !MotionEventCompat.isFromSource(ev, InputDevice.SOURCE_MOUSE)) {
		                  pullGlows(ev.getX(), unconsumedX, ev.getY(), unconsumedY);
		              }
		              considerReleasingGlowsOnScroll(x, y);
		          }
		          if (consumedX != 0 || consumedY != 0) {
		              dispatchOnScrolled(consumedX, consumedY);
		          }
		          if (!awakenScrollBars()) {
		              invalidate();
		          }
		          return consumedNestedScroll || consumedX != 0 || consumedY != 0;
		      }
		  
		  ```
	- ## 3、scrollStep
	  collapsed:: true
		- ```java
		      void scrollStep(int dx, int dy, @Nullable int[] consumed) {
		          startInterceptRequestLayout();
		          onEnterLayoutOrScroll();
		  
		          TraceCompat.beginSection(TRACE_SCROLL_TAG);
		          fillRemainingScrollValues(mState);
		  
		          int consumedX = 0;
		          int consumedY = 0;
		          // 水平滑动
		          if (dx != 0) {
		              consumedX = mLayout.scrollHorizontallyBy(dx, mRecycler, mState);
		          }
		          // 竖直滑动
		          if (dy != 0) {
		              consumedY = mLayout.scrollVerticallyBy(dy, mRecycler, mState);
		          }
		  
		          TraceCompat.endSection();
		          repositionShadowingViews();
		  
		          onExitLayoutOrScroll();
		          stopInterceptRequestLayout(false);
		  
		          if (consumed != null) {
		              consumed[0] = consumedX;
		              consumed[1] = consumedY;
		          }
		      }
		  
		  ```
- # LinearLayoutManager
  collapsed:: true
	- ## 4、先看个竖直滑动的LinearLayoutManager - scrollVerticallyBy
	  collapsed:: true
		- ```java
		      public int scrollVerticallyBy(int dy, RecyclerView.Recycler recycler,
		              RecyclerView.State state) {
		          if (mOrientation == HORIZONTAL) {
		              return 0;
		          }
		          return scrollBy(dy, recycler, state);
		      }
		  ```
	- ## 5、scrollBy
	  collapsed:: true
		- ```java
		  nt scrollBy(int delta, RecyclerView.Recycler recycler, RecyclerView.State state) {
		          if (getChildCount() == 0 || delta == 0) {
		              return 0;
		          }
		          ensureLayoutState();
		          mLayoutState.mRecycle = true;
		          final int layoutDirection = delta > 0 ? LayoutState.LAYOUT_END : LayoutState.LAYOUT_START;
		          final int absDelta = Math.abs(delta);
		          updateLayoutState(layoutDirection, absDelta, true, state);
		          final int consumed = mLayoutState.mScrollingOffset
		                  + fill(recycler, mLayoutState, state, false);
		          if (consumed < 0) {
		              if (DEBUG) {
		                  Log.d(TAG, "Don't have any more elements to scroll");
		              }
		              return 0;
		          }
		          final int scrolled = absDelta > consumed ? layoutDirection * consumed : delta;
		          mOrientationHelper.offsetChildren(-scrolled);
		          if (DEBUG) {
		              Log.d(TAG, "scroll req: " + delta + " scrolled: " + scrolled);
		          }
		          mLayoutState.mLastScrollDelta = scrolled;
		          return scrolled;
		      }
		  ```
	- ## 6、fill- 填充展现的布局-回收复用基本上在这里处理的
	  collapsed:: true
		- ```java
		  // 填充展现的布局
		  int fill(RecyclerView.Recycler recycler, LayoutState layoutState,
		              RecyclerView.State state, boolean stopOnFocusable) {
		          // max offset we should set is mFastScroll + available
		          final int start = layoutState.mAvailable;
		          if (layoutState.mScrollingOffset != LayoutState.SCROLLING_OFFSET_NaN) {
		              // TODO ugly bug fix. should not happen
		              if (layoutState.mAvailable < 0) {
		                  layoutState.mScrollingOffset += layoutState.mAvailable;
		              }
		              recycleByLayoutState(recycler, layoutState);
		          }
		          int remainingSpace = layoutState.mAvailable + layoutState.mExtraFillSpace;
		          LayoutChunkResult layoutChunkResult = mLayoutChunkResult;
		          while ((layoutState.mInfinite || remainingSpace > 0) && layoutState.hasMore(state)) {
		              layoutChunkResult.resetInternal();
		              if (RecyclerView.VERBOSE_TRACING) {
		                  TraceCompat.beginSection("LLM LayoutChunk");
		              }
		              layoutChunk(recycler, state, layoutState, layoutChunkResult);
		              if (RecyclerView.VERBOSE_TRACING) {
		                  TraceCompat.endSection();
		              }
		              if (layoutChunkResult.mFinished) {
		                  break;
		              }
		              layoutState.mOffset += layoutChunkResult.mConsumed * layoutState.mLayoutDirection;
		              /**
		               * Consume the available space if:
		               * * layoutChunk did not request to be ignored
		               * * OR we are laying out scrap children
		               * * OR we are not doing pre-layout
		               */
		              if (!layoutChunkResult.mIgnoreConsumed || layoutState.mScrapList != null
		                      || !state.isPreLayout()) {
		                  layoutState.mAvailable -= layoutChunkResult.mConsumed;
		                  // we keep a separate remaining space because mAvailable is important for recycling
		                  remainingSpace -= layoutChunkResult.mConsumed;
		              }
		  
		              if (layoutState.mScrollingOffset != LayoutState.SCROLLING_OFFSET_NaN) {
		                  layoutState.mScrollingOffset += layoutChunkResult.mConsumed;
		                  if (layoutState.mAvailable < 0) {
		                      layoutState.mScrollingOffset += layoutState.mAvailable;
		                  }
		                  recycleByLayoutState(recycler, layoutState);
		              }
		              if (stopOnFocusable && layoutChunkResult.mFocusable) {
		                  break;
		              }
		          }
		          if (DEBUG) {
		              validateChildOrder();
		          }
		          return start - layoutState.mAvailable;
		      }
		  ```
	- ## 7、layoutChunk,在循环里
	  collapsed:: true
		- 1、 layoutState.next(recycler);获取view
		- 2、addView(view); 添加view
		- ```java
		  void layoutChunk(RecyclerView.Recycler recycler, RecyclerView.State state,
		              LayoutState layoutState, LayoutChunkResult result) {
		         // 获取 view
		          View view = layoutState.next(recycler);
		          if (view == null) {
		              if (DEBUG && layoutState.mScrapList == null) {
		                  throw new RuntimeException("received null view when unexpected");
		              }
		              // if we are laying out views in scrap, this may return null which means there is
		              // no more items to layout.
		              result.mFinished = true;
		              return;
		          }
		         // 添加view
		          RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) view.getLayoutParams();
		          if (layoutState.mScrapList == null) {
		              if (mShouldReverseLayout == (layoutState.mLayoutDirection
		                      == LayoutState.LAYOUT_START)) {
		                  addView(view);
		              } else {
		                  addView(view, 0);
		              }
		          } else {
		              if (mShouldReverseLayout == (layoutState.mLayoutDirection
		                      == LayoutState.LAYOUT_START)) {
		                  addDisappearingView(view);
		              } else {
		                  addDisappearingView(view, 0);
		              }
		          }
		          measureChildWithMargins(view, 0, 0);
		          result.mConsumed = mOrientationHelper.getDecoratedMeasurement(view);
		          int left, top, right, bottom;
		          if (mOrientation == VERTICAL) {
		              if (isLayoutRTL()) {
		                  right = getWidth() - getPaddingRight();
		                  left = right - mOrientationHelper.getDecoratedMeasurementInOther(view);
		              } else {
		                  left = getPaddingLeft();
		                  right = left + mOrientationHelper.getDecoratedMeasurementInOther(view);
		              }
		              if (layoutState.mLayoutDirection == LayoutState.LAYOUT_START) {
		                  bottom = layoutState.mOffset;
		                  top = layoutState.mOffset - result.mConsumed;
		              } else {
		                  top = layoutState.mOffset;
		                  bottom = layoutState.mOffset + result.mConsumed;
		              }
		          } else {
		              top = getPaddingTop();
		              bottom = top + mOrientationHelper.getDecoratedMeasurementInOther(view);
		  
		              if (layoutState.mLayoutDirection == LayoutState.LAYOUT_START) {
		                  right = layoutState.mOffset;
		                  left = layoutState.mOffset - result.mConsumed;
		              } else {
		                  left = layoutState.mOffset;
		                  right = layoutState.mOffset + result.mConsumed;
		              }
		          }
		          // We calculate everything with View's bounding box (which includes decor and margins)
		          // To calculate correct layout position, we subtract margins.
		          layoutDecoratedWithMargins(view, left, top, right, bottom);
		          if (DEBUG) {
		              Log.d(TAG, "laid out child at position " + getPosition(view) + ", with l:"
		                      + (left + params.leftMargin) + ", t:" + (top + params.topMargin) + ", r:"
		                      + (right - params.rightMargin) + ", b:" + (bottom - params.bottomMargin));
		          }
		          // Consume the available space if the view is not removed OR changed
		          if (params.isItemRemoved() || params.isItemChanged()) {
		              result.mIgnoreConsumed = true;
		          }
		          result.mFocusable = view.hasFocusable();
		      }
		  ```
	- ## [[#red]]==**8、layoutState.next 不断的获取view**==
	  collapsed:: true
		- ```java
		          View next(RecyclerView.Recycler recycler) {
		              if (mScrapList != null) {
		                  return nextViewFromScrapList();
		              }
		              final View view = recycler.getViewForPosition(mCurrentPosition);
		              mCurrentPosition += mItemDirection;
		              return view;
		          }
		  ```
- # Recyclerview
	- ## 9、getViewForPosition
		- ```java
		          @NonNull
		          public View getViewForPosition(int position) {
		              return getViewForPosition(position, false);
		          }
		          // 拿到viewHolder 后 直接获取他的itemView
		          View getViewForPosition(int position, boolean dryRun) {
		              return tryGetViewHolderForPositionByDeadline(position, dryRun, FOREVER_NS).itemView;
		          }
		  ```
	- ## 10、[[tryGetViewHolderForPositionByDeadline从缓存找ViewHolder]] 去从四级缓存里取，没有就创建
	-
- # 时序图
	-