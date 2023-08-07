# dispatchTouchEvent源码
collapsed:: true
	- ```java
	  public boolean dispatchTouchEvent(MotionEvent event) {
	          // If the event should be handled by accessibility focus first.
	          if (event.isTargetAccessibilityFocus()) {
	              // We don't have focus or no virtual descendant has it, do not handle the event.
	              if (!isAccessibilityFocusedViewOrHost()) {
	                  return false;
	              }
	              // We have focus and got the event, then use normal event dispatch.
	              event.setTargetAccessibilityFocus(false);
	          }
	  
	          boolean result = false;
	  
	          if (mInputEventConsistencyVerifier != null) {
	              mInputEventConsistencyVerifier.onTouchEvent(event, 0);
	          }
	  
	          final int actionMasked = event.getActionMasked();
	          if (actionMasked == MotionEvent.ACTION_DOWN) {
	              // Defensive cleanup for new gesture
	              stopNestedScroll();
	          }
	  
	          // 事件是安全的 
	          if (onFilterTouchEventForSecurity(event)) {
	              if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
	                  result = true;
	              }
	              //noinspection SimplifiableIfStatement
	              // 1、最先调用的地方，view设置了mOnTouchListener，先走它的ontouch
	              ListenerInfo li = mListenerInfo;
	              if (li != null && li.mOnTouchListener != null
	                      && (mViewFlags & ENABLED_MASK) == ENABLED
	                      && li.mOnTouchListener.onTouch(this, event)) {
	                  result = true;
	              }
	  
	              if (!result && onTouchEvent(event)) {
	                  result = true;
	              }
	          }
	  
	          if (!result && mInputEventConsistencyVerifier != null) {
	              mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
	          }
	  
	          // Clean up after nested scrolls if this is the end of a gesture;
	          // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
	          // of the gesture.
	          if (actionMasked == MotionEvent.ACTION_UP ||
	                  actionMasked == MotionEvent.ACTION_CANCEL ||
	                  (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
	              stopNestedScroll();
	          }
	  
	          return result;
	      }
	  ```
- 1、最先调用的地方，view设置了[[#red]]==**mOnTouchListener**==，先走它的onTouch
  collapsed:: true
	- 看他返回值 如果为true那么 走进判断，result = true。事件消费掉了
	- ```java
	              if (li != null && li.mOnTouchListener != null
	                      && (mViewFlags & ENABLED_MASK) == ENABLED
	                      && li.mOnTouchListener.onTouch(this, event)) { // && 调用了touch的返回值
	                  result = true;
	              }
	  
	  ```
- 2、result = false 下边才会走[[#red]]==**onTouchEvent**==(event)
  collapsed:: true
	- > result = false 两种情况
	     1、要么 view没有写setOnTouchListener
	     2、要么写了返回false 没有消费
	- ```java
	  if (!result && onTouchEvent(event)) {
	    result = true;
	  }
	  ```
- # [[onTouchEvent源码]]
	- 3、Action_Down,会判断==**onLongClickListener**==
	- 4、Action_UP判断如果设置了[[#red]]==**mOnClickListener**==，则会走onClick方法。同时设置result = true 告诉父容器消费了
	- >onClick只要走就会消费，onTouch是有返回值的，可以决定消费与否