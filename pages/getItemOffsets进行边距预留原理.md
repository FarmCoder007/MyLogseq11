## 2、Recyclerview.addItemDecoration
collapsed:: true
	- ```java
	          // 自定义分割线
	          recyclerView.addItemDecoration(new StarDecoration(this));
	  ```
	- RecyclerVIew
		- ```java
		      public void addItemDecoration(@NonNull ItemDecoration decor) {
		          addItemDecoration(decor, -1);
		      }
		  
		      public void addItemDecoration(@NonNull ItemDecoration decor, int index) {
		          if (mLayout != null) {
		              mLayout.assertNotInLayoutOrScroll("Cannot add item decoration during a scroll  or"
		                      + " layout");
		          }
		          if (mItemDecorations.isEmpty()) {
		              setWillNotDraw(false);
		          }
		          if (index < 0) {
		              mItemDecorations.add(decor);
		          } else {
		              mItemDecorations.add(index, decor);
		          }
		          markItemDecorInsetsDirty();
		          requestLayout();
		      }
		  ```
	- requestLayout后，会调用onMeasure，onLayout，onDraw。
	- 首先执行到measureChildWithMargins
		- ```java
		          
		          public void measureChildWithMargins(@NonNull View child, int widthUsed, int heightUsed) {
		              final LayoutParams lp = (LayoutParams) child.getLayoutParams();
		  
		              final Rect insets = mRecyclerView.getItemDecorInsetsForChild(child);
		              widthUsed += insets.left + insets.right;
		              heightUsed += insets.top + insets.bottom;
		  
		              final int widthSpec = getChildMeasureSpec(getWidth(), getWidthMode(),
		                      getPaddingLeft() + getPaddingRight()
		                              + lp.leftMargin + lp.rightMargin + widthUsed, lp.width,
		                      canScrollHorizontally());
		              final int heightSpec = getChildMeasureSpec(getHeight(), getHeightMode(),
		                      getPaddingTop() + getPaddingBottom()
		                              + lp.topMargin + lp.bottomMargin + heightUsed, lp.height,
		                      canScrollVertically());
		              if (shouldMeasureChild(child, widthSpec, heightSpec, lp)) {
		                  child.measure(widthSpec, heightSpec);
		              }
		          }
		  ```
	- getItemDecorInsetsForChild
		- ```java
		    Rect getItemDecorInsetsForChild(View child) {
		          final LayoutParams lp = (LayoutParams) child.getLayoutParams();
		          if (!lp.mInsetsDirty) {
		              return lp.mDecorInsets;
		          }
		  
		          if (mState.isPreLayout() && (lp.isItemChanged() || lp.isViewInvalid())) {
		              // changed/invalid items should not be updated until they are rebound.
		              return lp.mDecorInsets;
		          }
		          final Rect insets = lp.mDecorInsets;
		          insets.set(0, 0, 0, 0);
		          final int decorCount = mItemDecorations.size();
		          for (int i = 0; i < decorCount; i++) {
		              mTempRect.set(0, 0, 0, 0);
		              mItemDecorations.get(i).getItemOffsets(mTempRect, child, this, mState);
		              insets.left += mTempRect.left;
		              insets.top += mTempRect.top;
		              insets.right += mTempRect.right;
		              insets.bottom += mTempRect.bottom;
		          }
		          lp.mInsetsDirty = false;
		          return insets;
		      }
		  ```
- ## 3、这里会调用getItemOffsets，就是我们自定义分割线重写的那个
  collapsed:: true
	- 将自定义的 上下左右  赋值给insets
- ## 4、在布局的时候调用layoutDecoratedWithMargins
  collapsed:: true
	- ```java
	          public void layoutDecoratedWithMargins(@NonNull View child, int left, int top, int right,
	                  int bottom) {
	              final LayoutParams lp = (LayoutParams) child.getLayoutParams();
	              final Rect insets = lp.mDecorInsets;
	              child.layout(left + insets.left + lp.leftMargin, top + insets.top + lp.topMargin,
	                      right - insets.right - lp.rightMargin,
	                      bottom - insets.bottom - lp.bottomMargin);
	          }
	  ```
	- 绘制child的 时候，将我们的左右上下 加入了