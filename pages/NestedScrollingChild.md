#  NestedScrollingChild
	- 代码
	  collapsed:: true
		- ```java
		  public interface NestedScrollingChild {
		    /**
		    * 启用或禁用嵌套滚动的方法，设置为true，并且当前界面的View的层次结构是支持嵌套滚动的
		    * (也就是需要NestedScrollingParent嵌套NestedScrollingChild)，才会触发嵌套滚动。
		    * 一般这个方法内部都是直接代理给NestedScrollingChildHelper的同名方法即可
		    */
		    void setNestedScrollingEnabled(boolean enabled);
		    
		    /**
		    * 判断当前View是否支持嵌套滑动。一般也是直接代理给NestedScrollingChildHelper的同名方法即可
		    */
		    boolean isNestedScrollingEnabled();
		    
		    /**
		    * 表示view开始滚动了,一般是在ACTION_DOWN中调用，如果返回true则表示父布局支持嵌套滚动。
		    * 一般也是直接代理给NestedScrollingChildHelper的同名方法即可。这个时候正常情况会触发Parent的onStartNestedScroll()方法
		    */
		    boolean startNestedScroll(@ScrollAxis int axes);
		    
		    /**
		    * 一般是在事件结束比如ACTION_UP或者ACTION_CANCLE中调用,告诉父布局滚动结束。一般也是直接代理给NestedScrollingChildHelper的同名方法即可
		    */
		    void stopNestedScroll();
		    
		    /**
		    * 判断当前View是否有嵌套滑动的Parent。一般也是直接代理给NestedScrollingChildHelper的同名方法即可
		    */
		    boolean hasNestedScrollingParent();
		    
		    /**
		    * 在当前View消费滚动距离之后。通过调用该方法，把剩下的滚动距离传给父布局。如果当前没有发生嵌套滚动，或者不支持嵌套滚动，调用该方法也没啥用。
		    * 内部一般也是直接代理给NestedScrollingChildHelper的同名方法即可
		    * dxConsumed：被当前View消费了的水平方向滑动距离
		    * dyConsumed：被当前View消费了的垂直方向滑动距离
		    * dxUnconsumed：未被消费的水平滑动距离
		    * dyUnconsumed：未被消费的垂直滑动距离
		    * offsetInWindow：输出可选参数。如果不是null，该方法完成返回时，
		    * 会将该视图从该操作之前到该操作完成之后的本地视图坐标中的偏移量封装进该参数中，offsetInWindow[0]水平方向，offsetInWindow[1]垂直方向
		    * @return true：表示滚动事件分发成功,fasle: 分发失败
		    */
		    boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,
		            int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow);
		    
		    /**
		    * 在当前View消费滚动距离之前把滑动距离传给父布局。相当于把优先处理权交给Parent
		    * 内部一般也是直接代理给NestedScrollingChildHelper的同名方法即可。
		  * dx：当前水平方向滑动的距离
		  * dy：当前垂直方向滑动的距离
		  * consumed：输出参数，会将Parent消费掉的距离封装进该参数consumed[0]代表水平方向，consumed[1]代表垂直方向
		  * @return true：代表Parent消费了滚动距离
		    */
		    boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,
		            @Nullable int[] offsetInWindow);
		    
		    /**
		    *将惯性滑动的速度分发给Parent。内部一般也是直接代理给NestedScrollingChildHelper的同名方法即可
		  * velocityX：表示水平滑动速度
		  * velocityY：垂直滑动速度
		  * consumed：true：表示当前View消费了滑动事件，否则传入false
		  * @return true：表示Parent处理了滑动事件
		  */
		    boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed);
		    
		    /**
		    * 在当前View自己处理惯性滑动前，先将滑动事件分发给Parent,一般来说如果想自己处理惯性的滑动事件，
		    * 就不应该调用该方法给Parent处理。如果给了Parent并且返回true，那表示Parent已经处理了，自己就不应该再做处理。
		    * 返回false，代表Parent没有处理，但是不代表Parent后面就不用处理了
		    * @return true：表示Parent处理了滑动事件
		    */
		    boolean dispatchNestedPreFling(float velocityX, float velocityY);
		  }
		  
		  ```
		  
		  
		  ```java
		  public interface NestedScrollingChild2 extends NestedScrollingChild {
		  
		    boolean startNestedScroll(@ScrollAxis int axes, @NestedScrollType int type);
		  
		    void stopNestedScroll(@NestedScrollType int type);
		  
		    boolean hasNestedScrollingParent(@NestedScrollType int type);
		  
		    boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,
		            int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow,
		            @NestedScrollType int type);
		  
		    boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,
		            @Nullable int[] offsetInWindow, @NestedScrollType int type);
		  }
		  ```
		  
		  ```java
		  public interface NestedScrollingChild3 extends NestedScrollingChild2 {
		  
		    void dispatchNestedScroll(int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed,
		            @Nullable int[] offsetInWindow, @ViewCompat.NestedScrollType int type,
		            @NonNull int[] consumed);
		  }
		  
		  ```
	- dispatchNestedPreScroll：子view滑动前先分发给父view
		- ```java
		  /**
		    * 在当前View消费滚动距离之前把滑动距离传给父布局。相当于把优先处理权交给Parent
		    * 内部一般也是直接代理给NestedScrollingChildHelper的同名方法即可。
		  * dx：当前水平方向滑动的距离
		  * dy：当前垂直方向滑动的距离
		  * consumed：输出参数，会将Parent消费掉的距离封装进该参数consumed[0]代表水平方向，consumed[1]代表垂直方向
		  * @return true：代表Parent消费了滚动距离
		    */
		    boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,
		            @Nullable int[] offsetInWindow);
		  ```
- ## NestedScrollingChild2比child1 多增加了类型
	- touch-手指滑动
	- non_touch-惯性滑动