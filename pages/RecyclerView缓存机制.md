- # 前言
	- 原生列表视图RecyclerView，最重要的亮点是其巧妙灵活的视图缓存策略，使其在渲染性能上体现出明显优势，这篇文章从源码的角度深入理解缓存机制以及和ListView相比较，优势体现在何处
- # 缓存机制
	- 首先使用时涉及几个核心类，LayoutManager、Adapter、ViewHolder、Recycler之间的关系，如何产生的关联
	  分析RecyclerView的onMeasure和onLayout方法，RecyclerView的绘制交给了LayoutManager。以LinearLayoutManager为例
		- ```
		  View next(RecyclerView.Recycler recycler) {
		              if (mScrapList != null) {
		                  return nextViewFromScrapList();
		              }
		              final View view = recycler.getViewForPosition(mCurrentPosition);
		              mCurrentPosition += mItemDirection;
		              return view;
		          }
		  ```