- ```java
  public interface NestedScrollingParent {
      /**
      * 当NestedScrollingChild调用方法startNestedScroll()时,会调用该方法。主要就是通过返回值告诉系统是否需要对后续的滚动进行处理
      * child：该ViewParen的包含NestedScrollingChild的直接子View，如果只有一层嵌套，和target是同一个View
      * target：本次嵌套滚动的NestedScrollingChild
      * nestedScrollAxes：滚动方向
      * @return 
      * true:表示我需要进行处理，后续的滚动会触发相应的回到
      * false: 我不需要处理，后面也就不会进行相应的回调了
      */
      //child和target的区别，如果是嵌套两层如:Parent包含一个LinearLayout，LinearLayout里面才是NestedScrollingChild类型的View。这个时候，
      //child指向LinearLayout，target指向NestedScrollingChild；如果Parent直接就包含了NestedScrollingChild，
      //这个时候target和child都指向NestedScrollingChild
      boolean onStartNestedScroll(@NonNull View child, @NonNull View target, @ScrollAxis int axes);
  
      /**
      * 如果onStartNestedScroll()方法返回的是true的话,那么紧接着就会调用该方法.它是让嵌套滚动在开始滚动之前,
      * 让布局容器(viewGroup)或者它的父类执行一些配置的初始化的
      */
      void onNestedScrollAccepted(@NonNull View child, @NonNull View target, @ScrollAxis int axes);
  
      /**
      * 停止滚动了,当子view调用stopNestedScroll()时会调用该方法
      */
      void onStopNestedScroll(@NonNull View target);
  
      /**
      * 当子view调用dispatchNestedScroll()方法时,会调用该方法。也就是开始分发处理嵌套滑动了
      * dxConsumed：已经被target消费掉的水平方向的滑动距离
      * dyConsumed：已经被target消费掉的垂直方向的滑动距离
      * dxUnconsumed：未被tagert消费掉的水平方向的滑动距离
      * dyUnconsumed：未被tagert消费掉的垂直方向的滑动距离
      */
      void onNestedScroll(@NonNull View target, int dxConsumed, int dyConsumed,
              int dxUnconsumed, int dyUnconsumed);
  
      /**
      * 当子view调用dispatchNestedPreScroll()方法是,会调用该方法。也就是在NestedScrollingChild在处理滑动之前，
      * 会先将机会给Parent处理。如果Parent想先消费部分滚动距离，将消费的距离放入consumed
      * dx：水平滑动距离
      * dy：处置滑动距离
      * consumed：表示Parent要消费的滚动距离,consumed[0]和consumed[1]分别表示父布局在x和y方向上消费的距离.
      */
      void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed);
  
      /**
      * 你可以捕获对内部NestedScrollingChild的fling事件
      * velocityX：水平方向的滑动速度
      * velocityY：垂直方向的滑动速度
      * consumed：是否被child消费了
      * @return
      * true:则表示消费了滑动事件
      */
      boolean onNestedFling(@NonNull View target, float velocityX, float velocityY, boolean consumed);
  
      /**
      * 在惯性滑动距离处理之前，会调用该方法，同onNestedPreScroll()一样，也是给Parent优先处理的权利
      * target：本次嵌套滚动的NestedScrollingChild
      * velocityX：水平方向的滑动速度
      * velocityY：垂直方向的滑动速度
      * @return
      * true：表示Parent要处理本次滑动事件，Child就不要处理了
      */
      boolean onNestedPreFling(@NonNull View target, float velocityX, float velocityY);
  
      /**
      * 返回当前滑动的方向，一般直接通过NestedScrollingParentHelper.getNestedScrollAxes()返回即可
      */
      @ScrollAxis
      int getNestedScrollAxes();
  }
  ```
  
  ```java
  public interface NestedScrollingParent2 extends NestedScrollingParent {
      boolean onStartNestedScroll(@NonNull View child, @NonNull View target, @ScrollAxis int axes,
              @NestedScrollType int type);
  
      void onNestedScrollAccepted(@NonNull View child, @NonNull View target, @ScrollAxis int axes,
              @NestedScrollType int type);
  
      void onStopNestedScroll(@NonNull View target, @NestedScrollType int type);
  
      void onNestedScroll(@NonNull View target, int dxConsumed, int dyConsumed,
              int dxUnconsumed, int dyUnconsumed, @NestedScrollType int type);
  
      void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed,
              @NestedScrollType int type);
  
  }
  ```
  
  ```java
  public interface NestedScrollingParent3 extends NestedScrollingParent2 {
  
      void onNestedScroll(@NonNull View target, int dxConsumed, int dyConsumed, int dxUnconsumed,
              int dyUnconsumed, @ViewCompat.NestedScrollType int type, @NonNull int[] consumed);
  
  }
  ```