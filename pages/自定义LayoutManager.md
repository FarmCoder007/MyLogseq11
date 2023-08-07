# 1、继承 RecyclerView.LayoutManager。完成item布局层叠感
collapsed:: true
	- 必须要重写的方法
		- onLayoutChildren
			- 1、布局前先回收复用
				- ```java
				          // ViewHodler回收复用
				          detachAndScrapAttachedViews(recycler);
				  ```
		- generateDefaultLayoutParams
		  collapsed:: true
			- ```java
			      @Override
			      public RecyclerView.LayoutParams generateDefaultLayoutParams() {
			          return new RecyclerView.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT,
			                  ViewGroup.LayoutParams.WRAP_CONTENT);
			      }
			  ```
	- ## SlideCardLayoutManager代码
	  collapsed:: true
		- ```java
		  public class SlideCardLayoutManager extends RecyclerView.LayoutManager {
		  
		      @Override
		      public RecyclerView.LayoutParams generateDefaultLayoutParams() {
		          return new RecyclerView.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT,
		                  ViewGroup.LayoutParams.WRAP_CONTENT);
		      }
		  
		      // 布局
		      @Override
		      public void onLayoutChildren(RecyclerView.Recycler recycler, RecyclerView.State state) {
		  
		          // ViewHodler回收复用
		          detachAndScrapAttachedViews(recycler);
		  
		          int bottomPosition;
		          int itemCount = getItemCount();
		          // 最大显示4张，真实个数小于4个时，直接从0开始
		          if (itemCount < CardConfig.MAX_SHOW_COUNT) {
		              bottomPosition = 0;
		          } else {
		              // 布局了四张卡片 --- 4，5，6，7  这里执行了倒序。也可改成正序、7张最大显示4张
		              // 最底下那张 position为 4
		              bottomPosition = itemCount - CardConfig.MAX_SHOW_COUNT;
		          }
		  
		          for (int i = bottomPosition; i < itemCount; i++) {
		              // 复用
		              View view = recycler.getViewForPosition(i);
		              // 添加view
		              addView(view);
		              // 先测量下view 拿到宽高。测量出 itemView的宽高来
		              measureChildWithMargins(view, 0, 0);
		              // getDecoratedMeasuredWidth 是获取 insetsleft 和 insets.right + 宽度（就是加了分割线的宽度了）
		              // widthSpace = 屏幕宽度 - 测量view宽度
		              // 左侧绘制起始点：就是left = widthSpace/2
		              // top = heightSpace / 2
		              int widthSpace = getWidth() - getDecoratedMeasuredWidth(view);
		              int heightSpace = getHeight() - getDecoratedMeasuredHeight(view);
		  
		              // 布局 ---draw -- onDraw ,onDrawOver, onLayout
		              layoutDecoratedWithMargins(view, widthSpace / 2, heightSpace / 2,
		                      widthSpace / 2 + getDecoratedMeasuredWidth(view),
		                      heightSpace / 2 + getDecoratedMeasuredHeight(view));
		  
		              // 到此为止 所有的都压在一起了
		  
		              // 进行平移和缩放。显出层次，设计 一个 level
		              int level = itemCount - i - 1;
		              if (level > 0) { // 4-7 进行平移缩放
		                  if (level < CardConfig.MAX_SHOW_COUNT - 1) {
		                      // 平移
		                      view.setTranslationY(CardConfig.TRANS_Y_GAP * level);
		                      // 缩放
		                      view.setScaleX(1 - CardConfig.SCALE_GAP * level);
		                      view.setScaleY(1 - CardConfig.SCALE_GAP * level);
		                  } else {// 下边1-3进行重叠
		                      // 最下面的那个view 与前一个view 布局一样
		                      view.setTranslationY(CardConfig.TRANS_Y_GAP * (level - 1));
		                      view.setScaleX(1 - CardConfig.SCALE_GAP * (level - 1));
		                      view.setScaleY(1 - CardConfig.SCALE_GAP * (level - 1));
		                  }
		              }
		          }
		  
		  
		      }
		  }
		  
		  ```
- # 2、条目拖拽使用ItemTouchHelper
	- 1、需要创建callBack
	  collapsed:: true
		- ```java
		  public class SlideCallback extends ItemTouchHelper.SimpleCallback {
		  
		      private RecyclerView mRv;
		      private UniversalAdapter<SlideCardBean> adapter;
		      private List<SlideCardBean> mDatas;
		  
		      public SlideCallback(RecyclerView mRv,
		                           UniversalAdapter<SlideCardBean> adapter, List<SlideCardBean> mDatas) {
		          // 参数1 拖拽 参数2 滑动方向 四面八方 up| down| left | right 最终等于15
		          super(0, 15);
		          this.mRv = mRv;
		          this.adapter = adapter;
		          this.mDatas = mDatas;
		      }
		  
		      // drag  拖拽
		      @Override
		      public boolean onMove(@NonNull RecyclerView recyclerView, @NonNull RecyclerView.ViewHolder viewHolder, @NonNull RecyclerView.ViewHolder target) {
		          return false;
		      }
		  
		      // 滑动
		      @Override
		      public void onSwiped(@NonNull RecyclerView.ViewHolder viewHolder, int direction) {
		          // 这里数据少 先移除再添加到第一个位置。实际上移除了就行了数据多的话
		          SlideCardBean remove = mDatas.remove(viewHolder.getLayoutPosition());
		          mDatas.add(0, remove);
		          adapter.notifyDataSetChanged();// onMeasure, onlayout
		      }
		  
		      // onDraw
		  
		  
		      /**
		       *  onDraw 时调用拖动绘制
		       * @param c                 The canvas which RecyclerView is drawing its children
		       * @param recyclerView      The RecyclerView to which ItemTouchHelper is attached to
		       * @param viewHolder        The ViewHolder which is being interacted by the User or it was
		       *                          interacted and simply animating to its original position
		       * @param dX                The amount of horizontal displacement caused by user's action
		       * @param dY                The amount of vertical displacement caused by user's action
		       * @param actionState       The type of interaction on the View. Is either {@link
		       *                          #ACTION_STATE_DRAG} or {@link #ACTION_STATE_SWIPE}.
		       * @param isCurrentlyActive True if this view is currently being controlled by the user or
		       *                          false it is simply animating back to its original state.
		       */
		      @Override
		      public void onChildDraw(@NonNull Canvas c, @NonNull RecyclerView recyclerView, @NonNull RecyclerView.ViewHolder viewHolder, float dX, float dY, int actionState, boolean isCurrentlyActive) {
		          super.onChildDraw(c, recyclerView, viewHolder, dX, dY, actionState, isCurrentlyActive);
		  
		          // 超过 宽度的0.5 就移除item 否则回来
		          double maxDistance = recyclerView.getWidth() * 0.5f;
		          // 计算位置
		          double distance = Math.sqrt(dX * dX + dY * dY);
		          // 通过位置计算百分比
		          double fraction = distance / maxDistance;
		  
		          if (fraction > 1) {
		              fraction = 1;
		          }
		  
		          // 显示的个数  4个
		          int itemCount = recyclerView.getChildCount();
		  
		          for (int i = 0; i < itemCount; i++) {
		              View view = recyclerView.getChildAt(i);
		  
		              int level = itemCount - i - 1;
		  
		              if (level > 0) {
		                  if (level < CardConfig.MAX_SHOW_COUNT - 1) {
		                      view.setTranslationY((float) (CardConfig.TRANS_Y_GAP * level - fraction * CardConfig.TRANS_Y_GAP));
		                      view.setScaleX((float) (1 - CardConfig.SCALE_GAP * level + fraction * CardConfig.SCALE_GAP));
		                      view.setScaleY((float) (1 - CardConfig.SCALE_GAP * level + fraction * CardConfig.SCALE_GAP));
		                  }
		              }
		          }
		      }
		  
		      // 设置回弹 时间
		      @Override
		      public long getAnimationDuration(@NonNull RecyclerView recyclerView, int animationType, float animateDx, float animateDy) {
		          return 1500;
		      }
		  }
		  
		  ```
	- ## 将touchHelper和 callBack绑定
		- ```java
		    // 参数1 Recyclerview  adapter  数据      
		    SlideCallback slideCallback = new SlideCallback(rv, adapter, mDatas);
		          ItemTouchHelper itemTouchHelper = new ItemTouchHelper(slideCallback);
		          itemTouchHelper.attachToRecyclerView(rv);
		  ```