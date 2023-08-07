- 动画
	- [[ItemAnimator源码分析]]
- [[优化]]
- [ConcatAdapter](https://juejin.cn/post/7064856244125138952)
- # [[回收复用]]-就是缓存机制
- # [[ItemDecoration分割线]]-自定义吸顶
- # [[自定义LayoutManager]]-实现探探卡片左滑右滑
- # [[NotifyDataSetChanged]]
- # [[outRect]]
- # 常见报错
  collapsed:: true
	- 1、刷新数据源时报错recyclerView报IndexOutOfBoundsException解决办法
		- 原因：
			- 这是因为在刷新页面时，一般会清空之前的数据，然后再装填新的数据，并且在装填新数据完毕的时候NotifyItemRangeChanged
		- 解决方案：
			- ```java
			  Public  void  setItems(List<T>  newItems){
			       validateItems(newItem);
			       //先移除
			       int  startPosition=hasHeader()?1:0;是否有头布局
			       int preSize=this.items.size();
			  	 if(presize>0){
			     		 this.items.clear();
			      	notifyItemRangeRemoved(startPosition,preSize);  //必须调用
			  	}
			       //添加新数据
			       this.Items.addAll(newItems);
			       notifyItemRangeChanged(startPosition,newItems.size);  //必须调用
			  }
			  ```
-
- # [[RecyclerVIew-面试题]]