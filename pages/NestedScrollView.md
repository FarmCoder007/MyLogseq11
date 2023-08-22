#### view的事件分发机制

伪代码
```java
public boolean dispatchTouchEvent(MotionEvent ev){
  boolean consume = false;
  if (onInterceptTouchEvent(ev)){
      consume =  onTouchEvnet(ev);
  } else{
      consume = child.dispatchTouchEvent(ev);
  }
  return consume;
}
```
- #### View滑动冲突
  
  1. 外部滑动方向与内部滑动方向不一致
  > 解决方案：外部拦截法，当事件传递到父View时，父View需要处理此事件，就拦截，不处理此事件就不拦截
  
  2. 外部滑动方向与内部滑动方向一致；
  > 内部拦截法，当事件传递到父View时，父View都传递给子View，如果子View需要处理此事件就消耗掉，否则就交给父View处理。但是这种方法和Android事件分发不一致，需要配合 requestDisallowInterceptTouchEvent 方法才能正常工作
  
  3. 上面两种情况的嵌套。
  > 某个View一旦决定拦截，那么这一个事件序列都只能由它来处理，并且它的onInterceptTouchEvent不会再被调用
  > 解决方案： 解决这种滑动冲突，可以用NestedScrollingParent 和 NestedScrollingChild 来解决
  
  
  NestedScrollView和ScrollView类似，是一个支持滚动的控件。此外，它还同时支持作NestedScrollingParent或者NestedScrollingChild进行嵌套滚动操作。默认是启用嵌套滚动的
  
  ```java
  public class NestedScrollView extends FrameLayout implements NestedScrollingParent3,
        NestedScrollingChild3, ScrollingView {}
  ```
- ####  onInterceptTouchEvent
  
  NestedScrollView也是一个ViewGroup，根据Android的触摸事件分发机制，一般会进入到onInterceptTouchEvent(MotionEvent ev)进行拦截判断
  
  ```java
  @Override
  public boolean onInterceptTouchEvent(MotionEvent ev) {
      
        final int action = ev.getAction();
        if ((action == MotionEvent.ACTION_MOVE) && mIsBeingDragged) {
            // 如果正在move而且被定性为正在拖拽中，直接返回true，将MotionEvent交给自己的onTouchEvent去处理
            return true;
        }
  
        switch (action & MotionEvent.ACTION_MASK) {
            case MotionEvent.ACTION_MOVE: {
               
                final int activePointerId = mActivePointerId;
                if (activePointerId == INVALID_POINTER) {
                  
                    break;
                }
  
                final int pointerIndex = ev.findPointerIndex(activePointerId);
                if (pointerIndex == -1) {
                    Log.e(TAG, "Invalid pointerId=" + activePointerId
                            + " in onInterceptTouchEvent");
                    break;
                }
  
                final int y = (int) ev.getY(pointerIndex);
                final int yDiff = Math.abs(y - mLastMotionY);
                // 判断是垂直方向的滑动事件
                if (yDiff > mTouchSlop
                        && (getNestedScrollAxes() & ViewCompat.SCROLL_AXIS_VERTICAL) == 0) {// 如果垂直拖动距离大于mTouchSlop，就认定是正在scroll
                    //因为又垂直方向的滑动了，所以这个时候需要告诉父view不要拦截了，我自己处理，然后根据嵌套滑动的规则，在我自己处理之前，我会优先问下父view是不是需要消费滑动的距离，如果没有嵌套滑动机制的情况下，那么当子view处理滑动了，那么父view就没机会处理了，或者是父view处理了，就没办法给子view处理了
                    mIsBeingDragged = true;
                    mLastMotionY = y;
                    initVelocityTrackerIfNotExists();
                    mVelocityTracker.addMovement(ev);
                    mNestedYOffset = 0;
                    final ViewParent parent = getParent();
                    // 因为自己要处理 所以叫Parent不要拦截
                    if (parent != null) {// 如果认定了是scrollView滑动，则不让父类拦截，后续所有的MotionEvent都会有NestedScrollView去处理
                        parent.requestDisallowInterceptTouchEvent(true);
                    }
                }
                break;
            }
  
            case MotionEvent.ACTION_DOWN: {
                final int y = (int) ev.getY();
                //如果这个触摸的位置不是在子view里面就不处理了
                if (!inChild((int) ev.getX(), y)) {
                    mIsBeingDragged = false;
                    recycleVelocityTracker();
                    break;
                }
  
                mLastMotionY = y;
                mActivePointerId = ev.getPointerId(0);
  
                initOrResetVelocityTracker();
                mVelocityTracker.addMovement(ev);
  
                mScroller.computeScrollOffset();
                mIsBeingDragged = !mScroller.isFinished();
                // 这里就是开始嵌套滑动的地方了， 请格外关注下，因为startNestedScroll跟，因为它跟Behavior的一个成员函数重名
                startNestedScroll(ViewCompat.SCROLL_AXIS_VERTICAL, ViewCompat.TYPE_TOUCH);
                break;
            }
  
            case MotionEvent.ACTION_CANCEL:
            case MotionEvent.ACTION_UP:
                //抬起或者取消事件了 就要停止嵌套滑动
                mIsBeingDragged = false;
                mActivePointerId = INVALID_POINTER;
                recycleVelocityTracker();
                if (mScroller.springBack(getScrollX(), getScrollY(), 0, 0, 0, getScrollRange())) {
                    ViewCompat.postInvalidateOnAnimation(this);
                }
                // 停止嵌套滑动，前提是要作为NestedScrollingChild，stopNestedScroll跟Behavior的一个成员函数重名
                stopNestedScroll(ViewCompat.TYPE_TOUCH);
                break;
            case MotionEvent.ACTION_POINTER_UP:
                onSecondaryPointerUp(ev);
                break;
        }
    // 返回值也好理解：如果正在拖拽中，则返回true，告诉系统MotionEvent交给NestedScrollView的OnTouchEvent去处理
    // 如果没有拖拽，比如ACTION_DOWN、ACTION_UP内部Button点击的MotionEvent，返回false，MotionEvent传递给子View
        return mIsBeingDragged;
    }
  ```
  主要做了以下几件事情：
  1. 在ACTION_MOVE中：在需要处理的情况下，将mIsBeingDragged置为true，将事件传递给自己的onTouchEvent()方法进行处理
  2. 在ACTION_DOWN中：一个是判断是否需要拦截事件，二是在合适的时候调用startNestedScroll()方法
  3. 在ACTION_UP或者ACTION_CANCEL中：将mIsBeingDragged重置为false，然后调用stopNestedScroll()停止嵌套滑动
  
  
  ```java
  @Override
    public void requestDisallowInterceptTouchEvent(boolean disallowIntercept) {
  
        if (disallowIntercept == ((mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0)) {
            // We're already in this state, assume our ancestors are too
            return;
        }
  //1. 设置ViewGroup自己的FLAG_DISALLOW_INTERCEPT标志位为true。
        if (disallowIntercept) {
            mGroupFlags |= FLAG_DISALLOW_INTERCEPT;
        } else {
            mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;
        }
        // Pass it up to our parent
        if (mParent != null) {
  //2 递归设置ViewGroup其父View的FLAG_DISALLOW_INTERCEPT标志位为true           mParent.requestDisallowInterceptTouchEvent(disallowIntercept);
        }
    }
  ```
  
  ```java
  @Override
  public boolean dispatchTouchEvent(MotionEvent ev) {
            ......
            //判断ViewGroup的FLAG_DISALLOW_INTERCEPT标志位是否为true，如果为true，调用ViewGroup的onInterceptTouchEvent的方法，如果不是则不会走
            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                //disallowIntercept:不允许拦截
                //requestDisallowInterceptTouchEvent(true)则不拦截，false则拦截，默认是允许拦截的
                if (!disallowIntercept) {
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
            } else {
                // There are no touch targets and this action is not an initial down
                // so this view group continues to intercept touches.
                intercepted = true;
            }
            ......
    }
  ```
- #### onTouchEvent
  
  ```java
  @Override
  public boolean onTouchEvent(MotionEvent ev) {
        initVelocityTrackerIfNotExists();
  
        final int actionMasked = ev.getActionMasked();
  
        if (actionMasked == MotionEvent.ACTION_DOWN) {
         // 这个是一个很重要的参数，视差值初始化为0
            mNestedYOffset = 0;
        }
  
    // CoordinatorLayout和AppbarLayout视差滑动的时候，有悬停效果
    // mNestedYOffset记录的是悬停时候的scroll视差值
        MotionEvent vtev = MotionEvent.obtain(ev);
        vtev.offsetLocation(0, mNestedYOffset);
  
        switch (actionMasked) {
            case MotionEvent.ACTION_DOWN: {
                if (getChildCount() == 0) {
                    return false;
                }
                if ((mIsBeingDragged = !mScroller.isFinished())) {
                    final ViewParent parent = getParent();
                    if (parent != null) {// 通知父View不要拦截触摸事件
                        parent.requestDisallowInterceptTouchEvent(true);
                    }
                }
                // 如果是在惯性滑动中 停止滑动
                if (!mScroller.isFinished()) {
                    abortAnimatedScroll();
                }
  
                mLastMotionY = (int) ev.getY();
                mActivePointerId = ev.getPointerId(0);
                //1. 开启嵌套滑动
                startNestedScroll(ViewCompat.SCROLL_AXIS_VERTICAL, ViewCompat.TYPE_TOUCH);
                break;
            }
            case MotionEvent.ACTION_MOVE:
                final int activePointerIndex = ev.findPointerIndex(mActivePointerId);
                if (activePointerIndex == -1) {
                    Log.e(TAG, "Invalid pointerId=" + mActivePointerId + " in onTouchEvent");
                    break;
                }
  
                final int y = (int) ev.getY(activePointerIndex);
                // 手指滑动距离
                int deltaY = mLastMotionY - y;
                if (!mIsBeingDragged && Math.abs(deltaY) > mTouchSlop) {
                    final ViewParent parent = getParent();
                    if (parent != null) {
                        parent.requestDisallowInterceptTouchEvent(true);
                    }
                    // 给mIsBeingDragged 定性
                    mIsBeingDragged = true;
                    if (deltaY > 0) {//向上滑动
                        deltaY -= mTouchSlop;
                    } else {//向下滑动
                        deltaY += mTouchSlop;
                    }
                }
                if (mIsBeingDragged) {
                    //2. Start with nested pre scrolling
                    //mScrollConsumed 输出参数？
                    //看NestedScrollingParent的onNestedPreScroll
                    // 先分发给Parent进行预处理
                    // 祖先view会根据deltaY和mScrollOffset来决定是否消费这个touch事件
                    // 如果祖先view决定消费这个MotionEvent，会把结果写在mScrollConsumed和mScrollOffset中
                    if (dispatchNestedPreScroll(0, deltaY, mScrollConsumed, mScrollOffset,
                            ViewCompat.TYPE_TOUCH)) {
                        // 如果Parent消费了滑动距离 需要减去
                        deltaY -= mScrollConsumed[1];
                        mNestedYOffset += mScrollOffset[1];
                    }
  
                    // Scroll to follow the motion event
                    mLastMotionY = y - mScrollOffset[1];
  
                    final int oldY = getScrollY();
                    final int range = getScrollRange();
                    final int overscrollMode = getOverScrollMode();
                    boolean canOverscroll = overscrollMode == View.OVER_SCROLL_ALWAYS
                            || (overscrollMode == View.OVER_SCROLL_IF_CONTENT_SCROLLS && range > 0);
  
                    // Calling overScrollByCompat will call onOverScrolled, which
                    // calls onScrollChanged if applicable.
                    // 滑动自己
                    if (overScrollByCompat(0, deltaY, 0, getScrollY(), 0, range, 0,
                            0, true) && !hasNestedScrollingParent(ViewCompat.TYPE_TOUCH)) {
                        // Break our velocity if we hit a scroll barrier.
                        // 如果没有overScroll且没有支持nested功能的父View，速度追踪重置
                        mVelocityTracker.clear();
                    }
                    // 重新计算未消费的距离
                    final int scrolledDeltaY = getScrollY() - oldY;
                    final int unconsumedY = deltaY - scrolledDeltaY;
  
                    mScrollConsumed[1] = 0;
                    // 分发给Parent进行嵌套滚动
                    dispatchNestedScroll(0, scrolledDeltaY, 0, unconsumedY, mScrollOffset,
                            ViewCompat.TYPE_TOUCH, mScrollConsumed);
                    // 每一次拖动都需要NestedParentView去计算是否视差了
                    mLastMotionY -= mScrollOffset[1];
                    mNestedYOffset += mScrollOffset[1];
  
                    if (canOverscroll) {
                        deltaY -= mScrollConsumed[1];
                         // 如果Parent没有消费 并且可以滚动 继续处理
                        ensureGlows();
                        final int pulledToY = oldY + deltaY;
                        if (pulledToY < 0) {
                            EdgeEffectCompat.onPull(mEdgeGlowTop, (float) deltaY / getHeight(),
                                    ev.getX(activePointerIndex) / getWidth());
                            if (!mEdgeGlowBottom.isFinished()) {
                                mEdgeGlowBottom.onRelease();
                            }
                        } else if (pulledToY > range) {
                            EdgeEffectCompat.onPull(mEdgeGlowBottom, (float) deltaY / getHeight(),
                                    1.f - ev.getX(activePointerIndex)
                                            / getWidth());
                            if (!mEdgeGlowTop.isFinished()) {
                                mEdgeGlowTop.onRelease();
                            }
                        }
                        if (mEdgeGlowTop != null
                                && (!mEdgeGlowTop.isFinished() || !mEdgeGlowBottom.isFinished())) {
                            ViewCompat.postInvalidateOnAnimation(this);
                        }
                    }
                }
                break;
            case MotionEvent.ACTION_UP:
                final VelocityTracker velocityTracker = mVelocityTracker;
                velocityTracker.computeCurrentVelocity(1000, mMaximumVelocity);
                int initialVelocity = (int) velocityTracker.getYVelocity(mActivePointerId);
                if ((Math.abs(initialVelocity) >= mMinimumVelocity)) {
                    // 如果达到了惯性的速度 分发惯性滑动事件
                    //// 先给Parent看是否需要处理
                    if (!dispatchNestedPreFling(0, -initialVelocity)) {
                        dispatchNestedFling(0, -initialVelocity, true);
                        //自己处理
                        fling(-initialVelocity);
                    }
                } else if (mScroller.springBack(getScrollX(), getScrollY(), 0, 0, 0,
                        getScrollRange())) {
                    ViewCompat.postInvalidateOnAnimation(this);
                }
                mActivePointerId = INVALID_POINTER;
                // 这个方法里面会调用stopNestedScroll(ViewCompat.TYPE_TOUCH)
                endDrag();
                break;
            case MotionEvent.ACTION_CANCEL:
                if (mIsBeingDragged && getChildCount() > 0) {
                    if (mScroller.springBack(getScrollX(), getScrollY(), 0, 0, 0,
                            getScrollRange())) {
                        ViewCompat.postInvalidateOnAnimation(this);
                    }
                }
                mActivePointerId = INVALID_POINTER;
                endDrag();
                break;
            case MotionEvent.ACTION_POINTER_DOWN: {
                final int index = ev.getActionIndex();
                mLastMotionY = (int) ev.getY(index);
                mActivePointerId = ev.getPointerId(index);
                break;
            }
            case MotionEvent.ACTION_POINTER_UP:
                onSecondaryPointerUp(ev);
                mLastMotionY = (int) ev.getY(ev.findPointerIndex(mActivePointerId));
                break;
        }
  
        if (mVelocityTracker != null) {
            mVelocityTracker.addMovement(vtev);
        }
        vtev.recycle();
  
  // 这里始终返回true 所以Parent作为父View其实是进不了onTouchEvent()方法的
        return true;
    }
  ```
  首先在onTouchEvent只需以Child的身份考虑，因为当作为Child身份始终返回true了(只有子view的onTouchEvent返回false,父view的onTouchEvent方法才有机会执行的)，意味着Parent永远都进不了该方法
  1. ACTION_DOWN中：
  如果是惯性滑动的情况，停止滑动。同样是调用startNestedScroll(ViewCompat.SCROLL_AXIS_VERTICAL, ViewCompat.TYPE_TOUCH);开启嵌套滑动，所以Parent中的onStartNestedScroll()可能不止一次调用。但是多次调用有影响吗？没影响，在里面没有做具体滚动操作，只是做是否需要处理的判断而已
  2. ACTION_MOVE中：
  先通过调用dispatchNestedPreScroll()分发给Parent进行滚动处理。然后再通过overScrollByCompat()自己处理滚动事件，最后再计算一下未消费的距离，再通过dispatchNestedScroll()继续给Parent进行处理。同时根据返回值，判断Parent是否处理了，进行下一步操作
  在child.dispatchNestedScroll() -> childhelper.dispatchNestedScroll() -> childhelper.dispatchNestedScrollInternal() -> ViewParentCompat.onNestedScroll() -> parent.onNestedScroll() -> NestedScrollView#onNestedScroll()[注意这里其实是另外一个NestedScrollView]
  
  ```java
  private void onNestedScrollInternal(int dyUnconsumed, int type, @Nullable int[] consumed) {
        final int oldScrollY = getScrollY();
        //// 滚动自己 消费掉距离
        scrollBy(0, dyUnconsumed);
        final int myConsumed = getScrollY() - oldScrollY;
  
        if (consumed != null) {
            consumed[1] += myConsumed;
        }
        final int myUnconsumed = dyUnconsumed - myConsumed;
        // 继续分发给上一级
        mChildHelper.dispatchNestedScroll(0, myConsumed, 0, myUnconsumed, null, type, consumed);
    }
  ```
  3. ACTION_UP和ACTION_CANCEL中：
  在这两个case中，最后都调用了endDrag()
  
  ```java
  private void endDrag() {
        mIsBeingDragged = false;
  
        recycleVelocityTracker();
        //停止嵌套滑动
        stopNestedScroll(ViewCompat.TYPE_TOUCH);
  
        if (mEdgeGlowTop != null) {
            mEdgeGlowTop.onRelease();
            mEdgeGlowBottom.onRelease();
        }
    }
  ```
  另外在ACTION_UP中，在调用endDrag()之前，还会先处理惯性滑动，然后父view不处理就自己处理， dispatchNestedFling(0, -initialVelocity, true)，consumed传true了，不会走dispatchNestedFling，最后自己处理
  
  ```java
  @Override
  public boolean onNestedFling(
            @NonNull View target, float velocityX, float velocityY, boolean consumed) {
        if (!consumed) {
            dispatchNestedFling(0, velocityY, true);
            fling((int) velocityY);
            return true;
        }
        return false;
    }
  ```