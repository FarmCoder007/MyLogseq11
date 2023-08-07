## 事件分发：dispatchTouchEvent源码
collapsed:: true
	- ```java
	      @Override
	      public boolean dispatchTouchEvent(MotionEvent ev) {
	          // debug用
	          if (mInputEventConsistencyVerifier != null) {
	              mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
	          }
	  
	          // 放大镜等操作，辅助功能
	          if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
	              ev.setTargetAccessibilityFocus(false);
	          }
	          // 记录事件处不处理
	          boolean handled = false;
	           // 1、看事件是否有效：有效的才处理，无效的为屏幕模糊
	          if (onFilterTouchEventForSecurity(ev)) {
	              final int action = ev.getAction();
	              final int actionMasked = action & MotionEvent.ACTION_MASK;
	  
	              // 判断是不是新事件的开始。down
	              if (actionMasked == MotionEvent.ACTION_DOWN) {
	                  // 重置状态，清除所有东西
	                  cancelAndClearTouchTargets(ev);
	                  resetTouchState();
	              }
	  
	              // Check for interception.
	              // 2、判断是否拦截
	              final boolean intercepted;
	              // 如果是个手势 && 还没有分发  （mFirstTouchTarget这个是链表存放分发的对象）
	              if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) {
	                  // 3、重点  判断标记 看孩子有没有说是否不允许拦截它的事件。  
	                  // 这个地方：可以给孩子提供机会，子view请求父亲不要拦截的
	                  final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
	                  // false 的话就是没说不让拦截
	                  if (!disallowIntercept) {
	                      // 再问 自己要不要拦截onInterceptTouchEvent
	                      intercepted = onInterceptTouchEvent(ev);
	                      ev.setAction(action); // restore action in case it was changed
	                  } else {
	                      intercepted = false;
	                  }
	              } else {
	                  // There are no touch targets and this action is not an initial down
	                  // so this view group continues to intercept touches.
	                  intercepted = true;
	              }
	  
	              // If intercepted, start normal event dispatch. Also if there is already
	              // a view that is handling the gesture, do normal event dispatch.
	              if (intercepted || mFirstTouchTarget != null) {
	                  ev.setTargetAccessibilityFocus(false);
	              }
	  
	              // Check for cancelation.
	              final boolean canceled = resetCancelNextUpFlag(this)
	                      || actionMasked == MotionEvent.ACTION_CANCEL;
	  
	              // Update list of touch targets for pointer down, if needed.
	              final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
	              TouchTarget newTouchTarget = null;
	              boolean alreadyDispatchedToNewTouchTarget = false;
	  ```
	- ```java
	  		   if (!canceled && !intercepted) {
	  
	                  // If the event is targeting accessibility focus we give it to the
	                  // view that has accessibility focus and if it does not handle it
	                  // we clear the flag and dispatch the event to all children as usual.
	                  // We are looking up the accessibility focused host to avoid keeping
	                  // state since these events are very rare.
	                  View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
	                          ? findChildWithAccessibilityFocus() : null;
	                  // 手势的开始
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
	                          // 遍历 孩子
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
	                              // 有人说它处理
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
	                                  // 就添加到 TouchTarge 记录已分发了，记录这个view
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
	- ```java
	  			 // Dispatch to touch targets.
	              if (mFirstTouchTarget == null) {
	                  // No touch targets so treat this as an ordinary view.
	                  handled = dispatchTransformedTouchEvent(ev, canceled, null,
	                          TouchTarget.ALL_POINTER_IDS);
	              } else {
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
	              }
	  
	              // Update list of touch targets for pointer up or cancel, if needed.
	              if (canceled
	                      || actionMasked == MotionEvent.ACTION_UP
	                      || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
	                  resetTouchState();
	              } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
	                  final int actionIndex = ev.getActionIndex();
	                  final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
	                  removePointersFromTouchTargets(idBitsToRemove);
	              }
	          }
	  
	          if (!handled && mInputEventConsistencyVerifier != null) {
	              mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
	          }
	          return handled;
	      }
	  ```
-