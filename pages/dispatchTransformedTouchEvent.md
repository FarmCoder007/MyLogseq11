## 源码
collapsed:: true
	- ```java
	    private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
	              View child, int desiredPointerIdBits) {
	          final boolean handled;
	  
	          // Canceling motions is a special case.  We don't need to perform any transformations
	          // or filtering.  The important part is the action, not the contents.
	          final int oldAction = event.getAction();
	          if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
	              event.setAction(MotionEvent.ACTION_CANCEL);
	              if (child == null) {
	                  handled = super.dispatchTouchEvent(event);
	              } else {
	                  handled = child.dispatchTouchEvent(event);
	              }
	              event.setAction(oldAction);
	              return handled;
	          }
	  
	          // Calculate the number of pointers to deliver.
	          final int oldPointerIdBits = event.getPointerIdBits();
	          final int newPointerIdBits = oldPointerIdBits & desiredPointerIdBits;
	  
	          // If for some reason we ended up in an inconsistent state where it looks like we
	          // might produce a motion event with no pointers in it, then drop the event.
	          if (newPointerIdBits == 0) {
	              return false;
	          }
	  
	          // If the number of pointers is the same and we don't need to perform any fancy
	          // irreversible transformations, then we can reuse the motion event for this
	          // dispatch as long as we are careful to revert any changes we make.
	          // Otherwise we need to make a copy.
	          final MotionEvent transformedEvent;
	          if (newPointerIdBits == oldPointerIdBits) {
	              if (child == null || child.hasIdentityMatrix()) {
	                  if (child == null) {
	                      handled = super.dispatchTouchEvent(event);
	                  } else {
	                      final float offsetX = mScrollX - child.mLeft;
	                      final float offsetY = mScrollY - child.mTop;
	                      event.offsetLocation(offsetX, offsetY);
	  
	                      handled = child.dispatchTouchEvent(event);
	  
	                      event.offsetLocation(-offsetX, -offsetY);
	                  }
	                  return handled;
	              }
	              transformedEvent = MotionEvent.obtain(event);
	          } else {
	              transformedEvent = event.split(newPointerIdBits);
	          }
	  
	          // Perform any necessary transformations and dispatch.
	          if (child == null) {
	              handled = super.dispatchTouchEvent(transformedEvent);
	          } else {
	              final float offsetX = mScrollX - child.mLeft;
	              final float offsetY = mScrollY - child.mTop;
	              transformedEvent.offsetLocation(offsetX, offsetY);
	              if (! child.hasIdentityMatrix()) {
	                  transformedEvent.transform(child.getInverseMatrix());
	              }
	  
	              handled = child.dispatchTouchEvent(transformedEvent);
	          }
	  
	          // Done.
	          transformedEvent.recycle();
	          return handled;
	      }
	  ```
- ## 首先viewGroup的拦截后 传入的child = null，看走向
	- ```java
	  if (child == null) {
	    handled = super.dispatchTouchEvent(transformedEvent);
	  }
	  ```
	- 走的super.dispatchTouchEvent，父类的就是view.java。还是走view的分发那一套。如果viewGroup 重写了 onTouch,onTouchEvent，等方法就会进入，判断是否消费
- ## 不拦截分发时(down 和 move都可走这)，找到能接受事件的子view。传入
	- ```java
	   else {
	              final float offsetX = mScrollX - child.mLeft;
	              final float offsetY = mScrollY - child.mTop;
	              transformedEvent.offsetLocation(offsetX, offsetY);
	              if (! child.hasIdentityMatrix()) {
	                  transformedEvent.transform(child.getInverseMatrix());
	              }
	  
	              handled = child.dispatchTouchEvent(transformedEvent);
	          }
	  ```
	- 走的child 的dispatchTouchEvent。
		- 如果child是viewGroup。那么viewGroup的 dispatchTouchEvent整个流程再走一遍
		- 如果是view,那么进入view的dispatchTouchEvent，看touchListener,onTouchevent  等