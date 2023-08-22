- ```java
  /**
  * 一个标准的嵌套滚动的框架策略,即如何控制了嵌套滚动的事件分发和一些逻辑处理
  * 就是在当前Child的所有的祖辈ViewParent中勋在一个实现了NestedScroolingParent接口，并且支持嵌套滚动(onStartNestedScroll()返回true)的。找到之后，在对应的分发方法中，将相关参数分发到ViewParent中与之对应的处理方法中。而且为了兼容性，都是通过ViewParentCompat进行转发操作的
  */
  public class NestedScrollingChildHelper {
      private ViewParent mNestedScrollingParentTouch;
      private ViewParent mNestedScrollingParentNonTouch;
      private final View mView;
      private boolean mIsNestedScrollingEnabled;
      private int[] mTempNestedScrollConsumed;
  
   
  	//就是传一个View进来，该View就是实现了NestedScrollingChild接口的View类型
      public NestedScrollingChildHelper(@NonNull View view) {
          mView = view;
      }
  
      public void setNestedScrollingEnabled(boolean enabled) {
      //主要用于是给变量mIsNestedScrollingEnabled进行赋值，
      //记录否可以支持嵌套滚动的方式。可以看到，如果之前是支持嵌套滚动的话，
      //会先调用ViewCompat.stopNestedScroll(mView)停止当前滚动，然后进行赋值操作
          if (mIsNestedScrollingEnabled) {
              ViewCompat.stopNestedScroll(mView);
          }
          mIsNestedScrollingEnabled = enabled;
      }
  
      public boolean isNestedScrollingEnabled() {
      //判断当前View是否支持嵌套滚动
          return mIsNestedScrollingEnabled;
      }
  
  
      public boolean hasNestedScrollingParent() {
          return hasNestedScrollingParent(TYPE_TOUCH);
      }
  
      public boolean hasNestedScrollingParent(@NestedScrollType int type) {
      //获取嵌套滚动的Parent，也就是实现了NestedScrollingParent的ViewGroup
          return getNestedScrollingParentForType(type) != null;
      }
  
      public boolean startNestedScroll(@ScrollAxis int axes) {
          return startNestedScroll(axes, TYPE_TOUCH);
      }
  
      public boolean startNestedScroll(@ScrollAxis int axes, @NestedScrollType int type) {
          if (hasNestedScrollingParent(type)) {
              // 先判断是否已经在嵌套滑动中，是的话，不处理
              return true;
          }
          if (isNestedScrollingEnabled()) {// 判断是否支持嵌套滑动
              ViewParent p = mView.getParent();
              View child = mView;
  // 这里就是利用循环一层一层的往上取出ParentView，直到该ParentView是支持嵌套滑动或者为null的时候
              while (p != null) {
  //就是判断当前ViewParent是否支持嵌套滑动。同时如果ViewParent是NestedScrollingParent的子类的话，会调用onStartNestedScroll()判断当前ViewParent是否需要嵌套滑动
                  //如果外层View有一个CoordinatorLayout，则这个NestedScrollView就能关联上CoordinatorLayout了
                  if (ViewParentCompat.onStartNestedScroll(p, child, mView, axes, type)) {//1
  // 给mNestedScrollingParentTouch赋值，后面就可以直接获取有效ViewParent
                      setNestedScrollingParentForType(type, p);
                      //注释1 返回true，注释②里面就会调用NestedScrollingParent的onNestedScrollAccepted()方法,也就是说如果要处理嵌套滑动，onStartNestedScroll必须返回true
                      ViewParentCompat.onNestedScrollAccepted(p, child, mView, axes, type);
                      return true;
                  }
                  if (p instanceof View) {
                      child = (View) p;
                  }
                  p = p.getParent();
              }
          }
          return false;
      }
  
      public void stopNestedScroll() {
          stopNestedScroll(TYPE_TOUCH);
      }
  
      public void stopNestedScroll(@NestedScrollType int type) {
          ViewParent parent = getNestedScrollingParentForType(type);
          if (parent != null) {
          // 这里面就会调用ViewParent的onStopNestedScroll(target, type)方法
              ViewParentCompat.onStopNestedScroll(parent, mView, type);
              // 该次滑动结束 将给mNestedScrollingParentTouch置空
              setNestedScrollingParentForType(type, null);
          }
      }
  
      public boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,
              int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow) {
          return dispatchNestedScrollInternal(dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed,
                  offsetInWindow, TYPE_TOUCH, null);
      }
  
  
      public boolean dispatchNestedScroll(int dxConsumed, int dyConsumed, int dxUnconsumed,
              int dyUnconsumed, @Nullable int[] offsetInWindow, @NestedScrollType int type) {
          return dispatchNestedScrollInternal(dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed,
                  offsetInWindow, type, null);
      }
  
  
      public void dispatchNestedScroll(int dxConsumed, int dyConsumed, int dxUnconsumed,
              int dyUnconsumed, @Nullable int[] offsetInWindow, @NestedScrollType int type,
              @Nullable int[] consumed) {
          dispatchNestedScrollInternal(dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed,
                  offsetInWindow, type, consumed);
      }
  
      private boolean dispatchNestedScrollInternal(int dxConsumed, int dyConsumed,
              int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow,
              @NestedScrollType int type, @Nullable int[] consumed) {
          if (isNestedScrollingEnabled()) {
          // 在startNestedScroll()中进行了赋值操作，所以这里可以直接获取ViewParent了
              final ViewParent parent = getNestedScrollingParentForType(type);
              if (parent == null) {
                  return false;
              }
   // 判断是否是有效的嵌套滑动
              if (dxConsumed != 0 || dyConsumed != 0 || dxUnconsumed != 0 || dyUnconsumed != 0) {
                  int startX = 0;
                  int startY = 0;
                  if (offsetInWindow != null) {
                      mView.getLocationInWindow(offsetInWindow);
                      startX = offsetInWindow[0];
                      startY = offsetInWindow[1];
                  }
  
                  if (consumed == null) {
                      consumed = getTempNestedScrollConsumed();
                      consumed[0] = 0;
                      consumed[1] = 0;
                  }
   // 这里就会调用ViewParent的onNestedScroll()方法
                  ViewParentCompat.onNestedScroll(parent, mView,
                          dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed, type, consumed);
  
                  if (offsetInWindow != null) {
                      mView.getLocationInWindow(offsetInWindow);
                      offsetInWindow[0] -= startX;
                      offsetInWindow[1] -= startY;
                  }
                  return true;
              } else if (offsetInWindow != null) {
                  offsetInWindow[0] = 0;
                  offsetInWindow[1] = 0;
              }
          }
          return false;
      }
  
  
      public boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,
              @Nullable int[] offsetInWindow) {
          return dispatchNestedPreScroll(dx, dy, consumed, offsetInWindow, TYPE_TOUCH);
      }
  
      public boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,
              @Nullable int[] offsetInWindow, @NestedScrollType int type) {
          if (isNestedScrollingEnabled()) {
              final ViewParent parent = getNestedScrollingParentForType(type);
              if (parent == null) {
                  return false;
              }
  
              if (dx != 0 || dy != 0) {
                  int startX = 0;
                  int startY = 0;
                  if (offsetInWindow != null) {
                      mView.getLocationInWindow(offsetInWindow);
                      startX = offsetInWindow[0];
                      startY = offsetInWindow[1];
                  }
  
                  if (consumed == null) {
                      consumed = getTempNestedScrollConsumed();
                  }
                  consumed[0] = 0;
                  consumed[1] = 0;
                  // 这里会调用ViewParent的onNestedPreScroll()方法 Parent消费的数据会缝在consumed变量中
                  ViewParentCompat.onNestedPreScroll(parent, mView, dx, dy, consumed, type);
  
                  if (offsetInWindow != null) {
                      mView.getLocationInWindow(offsetInWindow);
                      offsetInWindow[0] -= startX;
                      offsetInWindow[1] -= startY;
                  }
                  return consumed[0] != 0 || consumed[1] != 0;
              } else if (offsetInWindow != null) {
                  offsetInWindow[0] = 0;
                  offsetInWindow[1] = 0;
              }
          }
          return false;
      }
  
      public boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed) {
          if (isNestedScrollingEnabled()) {
              ViewParent parent = getNestedScrollingParentForType(TYPE_TOUCH);
              if (parent != null) {
              // 这里会调用ViewParent的onNestedFling()方法
                  return ViewParentCompat.onNestedFling(parent, mView, velocityX,
                          velocityY, consumed);
              }
          }
          return false;
      }
  
      public boolean dispatchNestedPreFling(float velocityX, float velocityY) {
          if (isNestedScrollingEnabled()) {
              ViewParent parent = getNestedScrollingParentForType(TYPE_TOUCH);
              if (parent != null) {
                  return ViewParentCompat.onNestedPreFling(parent, mView, velocityX,
                          velocityY);
              }
          }
          return false;
      }
  
      public void onDetachedFromWindow() {
          ViewCompat.stopNestedScroll(mView);
      }
  
      public void onStopNestedScroll(@NonNull View child) {
          ViewCompat.stopNestedScroll(mView);
      }
  
      private ViewParent getNestedScrollingParentForType(@NestedScrollType int type) {
          switch (type) {
              case TYPE_TOUCH:
                  return mNestedScrollingParentTouch;
              case TYPE_NON_TOUCH:
                  return mNestedScrollingParentNonTouch;
          }
          return null;
      }
  
      private void setNestedScrollingParentForType(@NestedScrollType int type, ViewParent p) {
          switch (type) {
              case TYPE_TOUCH:
                  mNestedScrollingParentTouch = p;
                  break;
              case TYPE_NON_TOUCH:
                  mNestedScrollingParentNonTouch = p;
                  break;
          }
      }
  
      private int[] getTempNestedScrollConsumed() {
          if (mTempNestedScrollConsumed == null) {
              mTempNestedScrollConsumed = new int[2];
          }
          return mTempNestedScrollConsumed;
      }
  }
  ```