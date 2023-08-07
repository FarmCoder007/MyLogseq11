## 调用源码的getChangedScrapViewForPosition
	- ```java
	          ViewHolder getChangedScrapViewForPosition(int position) {
	              // If pre-layout, check the changed scrap for an exact match.
	              final int changedScrapSize;
	              if (mChangedScrap == null || (changedScrapSize = mChangedScrap.size()) == 0) {
	                  return null;
	              }
	              // find by position
	              for (int i = 0; i < changedScrapSize; i++) {
	                  final ViewHolder holder = mChangedScrap.get(i);
	                  if (!holder.wasReturnedFromScrap() && holder.getLayoutPosition() == position) {
	                      holder.addFlags(ViewHolder.FLAG_RETURNED_FROM_SCRAP);
	                      return holder;
	                  }
	              }
	              // find by id
	              if (mAdapter.hasStableIds()) {
	                  final int offsetPosition = mAdapterHelper.findPositionOffset(position);
	                  if (offsetPosition > 0 && offsetPosition < mAdapter.getItemCount()) {
	                      final long id = mAdapter.getItemId(offsetPosition);
	                      for (int i = 0; i < changedScrapSize; i++) {
	                          final ViewHolder holder = mChangedScrap.get(i);
	                          if (!holder.wasReturnedFromScrap() && holder.getItemId() == id) {
	                              holder.addFlags(ViewHolder.FLAG_RETURNED_FROM_SCRAP);
	                              return holder;
	                          }
	                      }
	                  }
	              }
	              return null;
	          }
	  
	  ```
- ## mChangedScrap 主要与动画相关的
	- 1、根据id和 position从mChangedScrap 去拿