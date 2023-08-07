- 非取消 && 不拦截 走的代码
  collapsed:: true
	- ```java
	  if (!canceled && !intercepted) {
	  
	                  // If the event is targeting accessibility focus we give it to the
	                  // view that has accessibility focus and if it does not handle it
	                  // we clear the flag and dispatch the event to all children as usual.
	                  // We are looking up the accessibility focused host to avoid keeping
	                  // state since these events are very rare.
	                  View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
	                          ? findChildWithAccessibilityFocus() : null;
	  
	                  if (actionMasked == MotionEvent.ACTION_DOWN
	                          || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
	                          || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
	                      final int actionIndex = ev.getActionIndex(); // always 0 for down
	                      final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
	                              : TouchTarget.ALL_POINTER_IDS;
	  
	                      // Clean up earlier touch targets for this pointer id in case they
	                      // have become out of sync.
	                      removePointersFromTouchTargets(idBitsToAssign);
	  
	                      final int childrenCount = mChildrenCount;
	                      if (newTouchTarget == null && childrenCount != 0) {
	                          final float x = ev.getX(actionIndex);
	                          final float y = ev.getY(actionIndex);
	                          // Find a child that can receive the event.
	                          // Scan children from front to back.
	                          final ArrayList<View> preorderedList = buildTouchDispatchChildList();
	                          final boolean customOrder = preorderedList == null
	                                  && isChildrenDrawingOrderEnabled();
	                          final View[] children = mChildren;
	                          for (int i = childrenCount - 1; i >= 0; i--) {
	                              final int childIndex = getAndVerifyPreorderedIndex(
	                                      childrenCount, i, customOrder);
	                              final View child = getAndVerifyPreorderedView(
	                                      preorderedList, children, childIndex);
	  
	                              // If there is a view that has accessibility focus we want it
	                              // to get the event first and if not handled we will perform a
	                              // normal dispatch. We may do a double iteration but this is
	                              // safer given the timeframe.
	                              if (childWithAccessibilityFocus != null) {
	                                  if (childWithAccessibilityFocus != child) {
	                                      continue;
	                                  }
	                                  childWithAccessibilityFocus = null;
	                                  i = childrenCount - 1;
	                              }
	  
	                              if (!child.canReceivePointerEvents()
	                                      || !isTransformedTouchPointInView(x, y, child, null)) {
	                                  ev.setTargetAccessibilityFocus(false);
	                                  continue;
	                              }
	  
	                              newTouchTarget = getTouchTarget(child);
	                              if (newTouchTarget != null) {
	                                  // Child is already receiving touch within its bounds.
	                                  // Give it the new pointer in addition to the ones it is handling.
	                                  newTouchTarget.pointerIdBits |= idBitsToAssign;
	                                  break;
	                              }
	  
	                              resetCancelNextUpFlag(child);
	                              if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
	                                  // Child wants to receive touch within its bounds.
	                                  mLastTouchDownTime = ev.getDownTime();
	                                  if (preorderedList != null) {
	                                      // childIndex points into presorted list, find original index
	                                      for (int j = 0; j < childrenCount; j++) {
	                                          if (children[childIndex] == mChildren[j]) {
	                                              mLastTouchDownIndex = j;
	                                              break;
	                                          }
	                                      }
	                                  } else {
	                                      mLastTouchDownIndex = childIndex;
	                                  }
	                                  mLastTouchDownX = ev.getX();
	                                  mLastTouchDownY = ev.getY();
	                                  newTouchTarget = addTouchTarget(child, idBitsToAssign);
	                                  alreadyDispatchedToNewTouchTarget = true;
	                                  break;
	                              }
	  
	                              // The accessibility focus didn't handle the event, so clear
	                              // the flag and do a normal dispatch to all children.
	                              ev.setTargetAccessibilityFocus(false);
	                          }
	                          if (preorderedList != null) preorderedList.clear();
	                      }
	  
	                      if (newTouchTarget == null && mFirstTouchTarget != null) {
	                          // Did not find a child to receive the event.
	                          // Assign the pointer to the least recently added target.
	                          newTouchTarget = mFirstTouchTarget;
	                          while (newTouchTarget.next != null) {
	                              newTouchTarget = newTouchTarget.next;
	                          }
	                          newTouchTarget.pointerIdBits |= idBitsToAssign;
	                      }
	                  }
	              }
	  ```
- 1、遍历子view：找到能接受事件的view
	- 可见|| 执行animation动画
	- view 在点击范围内
- 2、调用 [[dispatchTransformedTouchEvent]] 这次传入的是找到能接受事件的子view
	- ### 不拦截分发时，找到能接受事件的子view，传入这个函数。内部调用child 的dispatchTouchEvent。
	  collapsed:: true
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
- 3、[[#red]]==**子view消费了**==:dispatchTransformedTouchEvent返回true代表，，会给mFirstTouchTarget赋值，就是这个view。alreadyDispatchedToNewTouchTarget = true
	- 然后viewGroup走向，看子view处理了，它就不处理了
	  collapsed:: true
		- ```java
		   // Dispatch to touch targets, excluding the new touch target if we already
		                  // dispatched to it.  Cancel touch targets if necessary.
		                  TouchTarget predecessor = null;
		                  TouchTarget target = mFirstTouchTarget;
		                  while (target != null) {
		                      final TouchTarget next = target.next;
		                      if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
		                          handled = true;
		                      } else {
		                          final boolean cancelChild = resetCancelNextUpFlag(target.child)
		                                  || intercepted;
		                          if (dispatchTransformedTouchEvent(ev, cancelChild,
		                                  target.child, target.pointerIdBits)) {
		                              handled = true;
		                          }
		                          if (cancelChild) {
		                              if (predecessor == null) {
		                                  mFirstTouchTarget = next;
		                              } else {
		                                  predecessor.next = next;
		                              }
		                              target.recycle();
		                              target = next;
		                              continue;
		                          }
		                      }
		                      predecessor = target;
		                      target = next;
		                  }
		  ```
- 4、所有子view全部不处理返回false的话，下边函数会按照拦截走
-