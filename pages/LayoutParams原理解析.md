- # LayoutParams
  collapsed:: true
	- LayoutParams翻译过来就是布局参数，子View通过LayoutParams告诉父容器（ViewGroup）应该如何放置自己。从这个定义中也可以看出来LayoutParams与ViewGroup是息息相关的，因此脱离ViewGroup谈LayoutParams是没有意义的。
	  事实上，每个ViewGroup的子类都有自己对应的LayoutParams类，典型的如LinearLayout.LayoutParams和
	  FrameLayout.LayoutParams等，可以看出来LayoutParams都是对应ViewGroup子类的内部类
- # MarginLayoutParams
  collapsed:: true
	- MarginLayoutParams是和外间距有关的。事实也确实如此，和LayoutParams相比，MarginLayoutParams只是增加了对上下左右外间距的支持。实际上大部分LayoutParams的实现类都是继承自MarginLayoutParams，因为基本所有的父容器都是支持子View设置外间距的
	- 属性优先级问题 MarginLayoutParams主要就是增加了上下左右4种外间距。在构造方法中，先是获取了
	  margin属性；如果该值不合法，就获取horizontalMargin；如果该值不合法，再去获取leftMargin和
	  rightMargin属性（verticalMargin、topMargin和bottomMargin同理）。我们可以据此总结出这几种属性的优先级
	- >margin > horizontalMargin和verticalMargin > leftMargin和RightMargin、topMargin和bottomMargin
	- 属性覆盖问题 优先级更高的属性会覆盖掉优先级较低的属性。此外，还要注意一下这几种属性上的注释
	- >Call {@link ViewGroup#setLayoutParams(LayoutParams)} after reassigning a new value
- # LayoutParams与View如何建立联系
  collapsed:: true
	- 在XML中定义View
	- 在Java代码中直接生成View对应的实例对象
- # addView四个重载方法
  collapsed:: true
	- ```java
	      /**
	       * 重载方法1：添加一个子View
	       * 如果这个子View还没有LayoutParams，就为子View设置当前ViewGroup默认的LayoutParams
	       */
	      public void addView(View child) {
	          addView(child, -1);
	      }
	      /**
	       * 重载方法2：在指定位置添加一个子View
	       * 如果这个子View还没有LayoutParams，就为子View设置当前ViewGroup默认的LayoutParams
	       * @param index View将在ViewGroup中被添加的位置（-1代表添加到末尾）
	       */
	      public void addView(View child, int index) {
	          if (child == null) {
	              throw new IllegalArgumentException("Cannot add a null child view to a ViewGroup");
	          }
	          LayoutParams params = child.getLayoutParams();
	          if (params == null) {
	              params = generateDefaultLayoutParams();// 生成当前ViewGroup默认的LayoutParams
	              if (params == null) {
	                  throw new IllegalArgumentException("generateDefaultLayoutParams() cannot return
	                  null");
	              }
	          }
	          addView(child, index, params);
	      }
	      /**
	       * 重载方法3：添加一个子View
	       * 使用当前ViewGroup默认的LayoutParams，并以传入参数作为LayoutParams的width和height
	       */
	      public void addView(View child, int width, int height) {
	          final LayoutParams params = generateDefaultLayoutParams(); // 生成当前ViewGroup默认的LayoutParams
	          params.width = width;
	          params.height = height;
	          addView(child, -1, params);
	      }
	      /**
	       * 重载方法4：添加一个子View，并使用传入的LayoutParams
	       */
	      @Override
	      public void addView(View child, LayoutParams params) {
	          addView(child, -1, params);
	      }
	      /**
	       * 重载方法4：在指定位置添加一个子View，并使用传入的LayoutParams
	       */
	      public void addView(View child, int index, LayoutParams params) {
	          if (child == null) {
	              throw new IllegalArgumentException("Cannot add a null child view to a ViewGroup");
	          }
	  
	          // addViewInner() will call child.requestLayout() when setting the new LayoutParams
	          // therefore, we call requestLayout() on ourselves before, so that the child's request
	          // will be blocked at our level
	          requestLayout();
	          invalidate(true);
	          addViewInner(child, index, params, false);
	      }
	      private void addViewInner(View child, int index, LayoutParams params,
	                                boolean preventRequestLayout) {
	          .....
	          if (mTransition != null) {
	              mTransition.addChild(this, child);
	          }
	          if (!checkLayoutParams(params)) { // ① 检查传入的LayoutParams是否合法
	              params = generateLayoutParams(params); // 如果传入的LayoutParams不合法，将进行转化操作
	          }
	          if (preventRequestLayout) { // ② 是否需要阻止重新执行布局流程
	              child.mLayoutParams = params; // 这不会引起子View重新布局（onMeasure->onLayout->onDraw）
	          } else {
	              child.setLayoutParams(params); // 这会引起子View重新布局（onMeasure->onLayout->onDraw）
	          }
	          if (index < 0) {
	              index = mChildrenCount;
	          }
	          addInArray(child, index);
	          // tell our children
	          if (preventRequestLayout) {
	              child.assignParent(this);
	          } else {
	              child.mParent = this;
	          }
	          .....
	      }
	  ```
- # 自定义LayoutParams
  collapsed:: true
	- ## 1. 创建自定义属性
	  collapsed:: true
		- ```xml
		      <resources>
		          <declare-styleable name="xxxViewGroup_Layout">
		              <!-- 自定义的属性 -->
		              <attr name="layout_simple_attr" format="integer"/>
		              <!-- 使用系统预置的属性 -->
		              <attr name="android:layout_gravity"/>
		          </declare-styleable>
		      </resources>
		  ```
	- ## 2. 继承MarginLayout
	  collapsed:: true
		- ```java
		  public static class LayoutParams extends ViewGroup.MarginLayoutParams {
		      public int simpleAttr;
		      public int gravity;
		      public LayoutParams(Context c, AttributeSet attrs) {
		          super(c, attrs);
		          // 解析布局属性
		          TypedArray typedArray = c.obtainStyledAttributes(attrs,
		                  R.styleable.SimpleViewGroup_Layout);
		          simpleAttr =
		                  typedArray.getInteger(R.styleable.SimpleViewGroup_Layout_layout_simple_attr, 0);
		          gravity=typedArray.getInteger(R.styleable.SimpleViewGroup_Layout_android_layout_gravity,
		                  -1);
		          typedArray.recycle();//释放资源
		      }
		      public LayoutParams(int width, int height) {
		          super(width, height);
		      }
		      public LayoutParams(MarginLayoutParams source) {
		          super(source);
		      }
		      public LayoutParams(ViewGroup.LayoutParams source) {
		          super(source);
		      }
		  }
		  
		  ```
	- ## 3. 重写ViewGroup中几个与LayoutParams相关的方法
	  collapsed:: true
		- ```java
		      // 检查LayoutParams是否合法
		      @Override
		      protected boolean checkLayoutParams(ViewGroup.LayoutParams p) {
		          return p instanceof SimpleViewGroup.LayoutParams;
		      }
		      // 生成默认的LayoutParams
		      @Override
		      protected ViewGroup.LayoutParams generateDefaultLayoutParams() {
		          return new SimpleViewGroup.LayoutParams(LayoutParams.MATCH_PARENT,
		                  LayoutParams.WRAP_CONTENT);
		      }
		      // 对传入的LayoutParams进行转化
		      @Override
		      protected ViewGroup.LayoutParams generateLayoutParams(ViewGroup.LayoutParams p) {
		          return new SimpleViewGroup.LayoutParams(p);
		      }
		      // 对传入的LayoutParams进行转化
		      @Override
		      public ViewGroup.LayoutParams generateLayoutParams(AttributeSet attrs) {
		          return new SimpleViewGroup.LayoutParams(getContext(), attrs);
		      }
		  ```
- # LayoutParams常见的子类
  collapsed:: true
	- 在为View设置LayoutParams的时候需要根据它的父容器选择对应的LayoutParams，否则结果可能与预期不一致这里简单罗列一些常见的LayoutParams子类：
	- ViewGroup.MarginLayoutParams
	  FrameLayout.LayoutParams
	  LinearLayout.LayoutParams
	  RelativeLayout.LayoutParams
	  RecyclerView.LayoutParams
	  GridLayoutManager.LayoutParams
	  StaggeredGridLayoutManager.LayoutParams
	  ViewPager.LayoutParams
	  WindowManager.LayoutParams