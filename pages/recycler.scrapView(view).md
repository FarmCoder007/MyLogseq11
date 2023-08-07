title:: recycler.scrapView(view)

- ## 源码
	- ```java
	  void scrapView(View view) {
	              final ViewHolder holder = getChildViewHolderInt(view);
	              if (holder.hasAnyOfTheFlags(ViewHolder.FLAG_REMOVED | ViewHolder.FLAG_INVALID)
	                      || !holder.isUpdated() || canReuseUpdatedViewHolder(holder)) {
	                  if (holder.isInvalid() && !holder.isRemoved() && !mAdapter.hasStableIds()) {
	                      throw new IllegalArgumentException("Called scrap view with an invalid view."
	                              + " Invalid views cannot be reused from scrap, they should rebound from"
	                              + " recycler pool." + exceptionLabel());
	                  }
	                  holder.setScrapContainer(this, false);
	                  mAttachedScrap.add(holder);
	              } else {
	                  if (mChangedScrap == null) {
	                      mChangedScrap = new ArrayList<ViewHolder>();
	                  }
	                  holder.setScrapContainer(this, true);
	                  mChangedScrap.add(holder);
	              }
	          }
	  
	  ```
- 1、判断条件：标记没有移除，失效，更新，动画不复用或者没有动画的时候
	- 通过mAttachedScrap 进行保存
- 2、否则
	- mChangedScrap 进行保存