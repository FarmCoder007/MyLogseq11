- NotifyDataSetChanged
	- ```java
	          public final void notifyDataSetChanged() {
	              mObservable.notifyChanged();
	          }
	  ```
- AdapterDataObservable-notifyChanged
	- ```java
	          public void notifyChanged() {
	              // since onChanged() is implemented by the app, it could do anything, including
	              // removing itself from {@link mObservers} - and that could cause problems if
	              // an iterator is used on the ArrayList {@link mObservers}.
	              // to avoid such problems, just march thru the list in the reverse order.
	              for (int i = mObservers.size() - 1; i >= 0; i--) {
	                  mObservers.get(i).onChanged();
	              }
	          }
	  ```
- RecyclerViewDataObserver:mObservers看其实现类，RecyclerView的实现类，
	- ```java
	   @Override
	          public void onChanged() {
	              assertNotInLayoutOrScroll(null);
	              mState.mStructureChanged = true;
	  
	              processDataSetCompletelyChanged(true);
	              if (!mAdapterHelper.hasPendingUpdates()) {
	                  requestLayout();
	              }
	          }
	  
	  ```
- requestLayout
- onMeaseru
- onLayout
- onDraw