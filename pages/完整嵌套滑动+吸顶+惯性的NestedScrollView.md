- ```java
  package com.xiangxue.nestedscroll.e_perfect_nestedscroll;
  
  import android.content.Context;
  import android.util.AttributeSet;
  import android.util.Log;
  import android.view.View;
  import android.view.ViewGroup;
  
  import androidx.annotation.NonNull;
  import androidx.annotation.Nullable;
  import androidx.core.widget.NestedScrollView;
  import androidx.recyclerview.widget.RecyclerView;
  
  import com.xiangxue.common.fragment.NestedLogRecyclerView;
  import com.xiangxue.common.utils.FlingHelper;
  
  public class NestedScrollLayout extends NestedScrollView {
      /**
       *  拿到tablayout  上边 那个header
       */
      private View topView;
      /**
       *  tablayout+viewpager
       */
      private ViewGroup contentView;
      private static final String TAG = "NestedScrollLayout";
  
      public NestedScrollLayout(Context context) {
          this(context, null);
          init();
      }
  
      public NestedScrollLayout(Context context, @Nullable AttributeSet attrs) {
          this(context, attrs, 0);
          init();
      }
  
      public NestedScrollLayout(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
          this(context, attrs, defStyleAttr, 0);
          init();
      }
  
      public NestedScrollLayout(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
          super(context, attrs, defStyleAttr);
          init();
      }
  
      private FlingHelper mFlingHelper;
  
      int totalDy = 0;
      /**
       * 用于判断RecyclerView是否在fling
       */
      boolean isStartFling = false;
      /**
       * 记录当前滑动的y轴加速度
       */
      private int velocityY = 0;
  
      private void init() {
          mFlingHelper = new FlingHelper(getContext());
          setOnScrollChangeListener(new View.OnScrollChangeListener() {
              @Override
              public void onScrollChange(View v, int scrollX, int scrollY, int oldScrollX, int oldScrollY) {
                  if (isStartFling) {
                      totalDy = 0;
                      isStartFling = false;
                  }
                  if (scrollY == 0) {
                      Log.i(TAG, "TOP SCROLL");
                     // refreshLayout.setEnabled(true);
                  }
                  // 父亲滑到header 底部 让 子view滑动
                  if (scrollY == (getChildAt(0).getMeasuredHeight() - v.getMeasuredHeight())) {
                      Log.i(TAG, "BOTTOM SCROLL");
                      // 开始让底部滑动
                      dispatchChildFling();
                  }
                  //在RecyclerView fling情况下，记录当前RecyclerView在y轴的偏移
                  totalDy += scrollY - oldScrollY;
              }
          });
      }
  
      /**
       *  分发孩子的惯性滑动
       */
      private void dispatchChildFling() {
          if (velocityY != 0) {
              // 根据速度得出 惯性距离
              Double splineFlingDistance = mFlingHelper.getSplineFlingDistance(velocityY);
              // 大于 父亲能滑动的距离
              if (splineFlingDistance > totalDy) {
                  // 惯性距离- 父亲滑动距离      以后再转换成速度，分发给孩子滑动
                  childFling(mFlingHelper.getVelocityByDistance(splineFlingDistance - Double.valueOf(totalDy)));
              }
          }
          totalDy = 0;
          velocityY = 0;
      }
  
      /**
       *  传入计算出来的速度。让孩子滑动
       * @param velY
       */
      private void childFling(int velY) {
          RecyclerView childRecyclerView = getChildRecyclerView(contentView);
          if (childRecyclerView != null) {
              childRecyclerView.fling(0, velY);
          }
      }
  
      /**
       * 惯性滑动的时候回调
       * @param velocityY The initial velocity in the Y direction. Positive
       *                  numbers mean that the finger/cursor is moving down the screen,
       *                  which means we want to scroll towards the top.
       */
      @Override
      public void fling(int velocityY) {
          super.fling(velocityY);
          if (velocityY <= 0) {
              this.velocityY = 0;
          } else {
              // 记录惯性滑动的速度
              isStartFling = true;
              this.velocityY = velocityY;
          }
      }
  
      /**
       *  拿到子view
       */
      @Override
      protected void onFinishInflate() {
          super.onFinishInflate();
          topView = ((ViewGroup) getChildAt(0)).getChildAt(0);
          contentView = (ViewGroup) ((ViewGroup) getChildAt(0)).getChildAt(1);
      }
  
      /**
       * 吸顶效果：设置 contentView为屏幕高度：
       * @param widthMeasureSpec horizontal space requirements as imposed by the parent.
       *                         The requirements are encoded with
       *                         {@link android.view.View.MeasureSpec}.
       * @param heightMeasureSpec vertical space requirements as imposed by the parent.
       *                         The requirements are encoded with
       *                         {@link android.view.View.MeasureSpec}.
       *
       */
      @Override
      protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
          // 调整contentView的高度为父容器高度，使之填充布局，避免父容器滚动后出现空白
          super.onMeasure(widthMeasureSpec, heightMeasureSpec);
          ViewGroup.LayoutParams lp = contentView.getLayoutParams();
          lp.height = getMeasuredHeight();
          contentView.setLayoutParams(lp);
      }
  
      /**
       *  子view 让我先滑动。我划不动它再滑
       * @param target View that initiated the nested scroll
       * @param dx Horizontal scroll distance in pixels
       * @param dy Vertical scroll distance in pixels
       * @param consumed Output. The horizontal and vertical scroll distance consumed by this parent
       *                 父亲消费的距离。消费完后剩下的 子view再滑动。这里 数组 0为x 1为y
       * @param type the type of input which cause this scroll event
       */
      @Override
      public void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed, int type) {
          Log.i("NestedScrollLayout", getScrollY()+"::onNestedPreScroll::"+topView.getMeasuredHeight());
          // 向上滑动。若当前topview可见，需要将topview滑动至不可见。即吸顶的时候我就不滑动了
          boolean hideTop = dy > 0 && getScrollY() < topView.getMeasuredHeight();
          if (hideTop) {
              // 滑动我自己
              scrollBy(0, dy);
              // 消费掉我滑动的距离，省的子view在我滑动的时候，它也滑动
              consumed[1] = dy;
          }
      }
  
      /**
       *  找到里层的RecyclerView
       * @param viewGroup
       * @return
       */
      private RecyclerView getChildRecyclerView(ViewGroup viewGroup) {
          for (int i = 0; i < viewGroup.getChildCount(); i++) {
              View view = viewGroup.getChildAt(i);
              if (view instanceof RecyclerView && view.getClass() == NestedLogRecyclerView.class) {
                  return (RecyclerView) viewGroup.getChildAt(i);
              } else if (viewGroup.getChildAt(i) instanceof ViewGroup) {
                  ViewGroup childRecyclerView = getChildRecyclerView((ViewGroup) viewGroup.getChildAt(i));
                  if (childRecyclerView instanceof RecyclerView) {
                      return (RecyclerView) childRecyclerView;
                  }
              }
              continue;
          }
          return null;
      }
  }
  
  ```