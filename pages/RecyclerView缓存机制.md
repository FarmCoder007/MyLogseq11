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
	- 核心方法执行路径：LinearLayoutManager.next方法 => recycleView.getViewForPosition=> 最终调用位置：tryGetViewHolderForPositionByDeadline
	- 通过分析源码可知，从两个维度分析得到结论：四级缓存，四次尝试
	  mAttachScrap：一级缓存，缓存屏幕内ViewHolder、mChangedScrap缓存变更ViewHolder
	  mCacheView：  二级缓存，可以理解为即将进入屏幕内的缓存，从mCacheView取用的holder不需要再次bind，默认缓存数量是2，验证时数量为3
	  mViewCacheExtension ：三级缓存，自定义缓存
	  mRecyclePool ：四级缓存，缓存复用需要重新bind，RecycleView 特性。和ListView不同的地方。支持多个RecyclerView之间复用ViewHolder
		- ![image.png](../assets/image_1684430178660_0.png){:height 1172, :width 716}
- ## 四次尝试
	- 第一次尝试:核心方法getScrapOrHiddenOrCachedHolderForPosition方法来获取ViewHolder，并检验holder的有效性，如果无效，则从mAttachedScrap中移除，并加入到mCacheViews或者Pool中，并且将holder至null，走下一级缓存判断
		- ⚠️：没有获取type
		  collapsed:: true
			- ![image.png](../assets/image_1684430207390_0.png)
	- 第二次尝试
		- 和第一次整体流程差不多，核心方法getScrapOrCachedViewForId，重点：内部调用了mAdapter.getItemViewType(offsetPosition)，多了对id和type的校验
		-