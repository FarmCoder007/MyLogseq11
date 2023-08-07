## 源码
collapsed:: true
	- ```java
	   public boolean onTouchEvent(MotionEvent event) {
	          final float x = event.getX();
	          final float y = event.getY();
	          final int viewFlags = mViewFlags;
	          final int action = event.getAction();
	  
	          final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
	                  || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
	                  || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;
	  
	          if ((viewFlags & ENABLED_MASK) == DISABLED) {
	              if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
	                  setPressed(false);
	              }
	              mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
	              // A disabled view that is clickable still consumes the touch
	              // events, it just doesn't respond to them.
	              return clickable;
	          }
	          if (mTouchDelegate != null) {
	              if (mTouchDelegate.onTouchEvent(event)) {
	                  return true;
	              }
	          }
	  
	          if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
	              switch (action) {
	                  case MotionEvent.ACTION_UP:
	                      mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
	                      if ((viewFlags & TOOLTIP) == TOOLTIP) {
	                          handleTooltipUp();
	                      }
	                      if (!clickable) {
	                          removeTapCallback();
	                          removeLongPressCallback();
	                          mInContextButtonPress = false;
	                          mHasPerformedLongPress = false;
	                          mIgnoreNextUpEvent = false;
	                          break;
	                      }
	                      boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
	                      if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
	                          // take focus if we don't have it already and we should in
	                          // touch mode.
	                          boolean focusTaken = false;
	                          if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
	                              focusTaken = requestFocus();
	                          }
	  
	                          if (prepressed) {
	                              // The button is being released before we actually
	                              // showed it as pressed.  Make it show the pressed
	                              // state now (before scheduling the click) to ensure
	                              // the user sees it.
	                              setPressed(true, x, y);
	                          }
	  
	                          if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
	                              // This is a tap, so remove the longpress check
	                              removeLongPressCallback();
	  
	                              // Only perform take click actions if we were in the pressed state
	                              if (!focusTaken) {
	                                  // Use a Runnable and post this rather than calling
	                                  // performClick directly. This lets other visual state
	                                  // of the view update before click actions start.
	                                  if (mPerformClick == null) {
	                                      mPerformClick = new PerformClick();
	                                  }
	                                  if (!post(mPerformClick)) {
	                                      performClickInternal();
	                                  }
	                              }
	                          }
	  
	                          if (mUnsetPressedState == null) {
	                              mUnsetPressedState = new UnsetPressedState();
	                          }
	  
	                          if (prepressed) {
	                              postDelayed(mUnsetPressedState,
	                                      ViewConfiguration.getPressedStateDuration());
	                          } else if (!post(mUnsetPressedState)) {
	                              // If the post failed, unpress right now
	                              mUnsetPressedState.run();
	                          }
	  
	                          removeTapCallback();
	                      }
	                      mIgnoreNextUpEvent = false;
	                      break;
	  
	                  case MotionEvent.ACTION_DOWN:
	                      if (event.getSource() == InputDevice.SOURCE_TOUCHSCREEN) {
	                          mPrivateFlags3 |= PFLAG3_FINGER_DOWN;
	                      }
	                      mHasPerformedLongPress = false;
	  
	                      if (!clickable) {
	                          checkForLongClick(
	                                  ViewConfiguration.getLongPressTimeout(),
	                                  x,
	                                  y,
	                                  TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__LONG_PRESS);
	                          break;
	                      }
	  
	                      if (performButtonActionOnTouchDown(event)) {
	                          break;
	                      }
	  
	                      // Walk up the hierarchy to determine if we're inside a scrolling container.
	                      boolean isInScrollingContainer = isInScrollingContainer();
	  
	                      // For views inside a scrolling container, delay the pressed feedback for
	                      // a short period in case this is a scroll.
	                      if (isInScrollingContainer) {
	                          mPrivateFlags |= PFLAG_PREPRESSED;
	                          if (mPendingCheckForTap == null) {
	                              mPendingCheckForTap = new CheckForTap();
	                          }
	                          mPendingCheckForTap.x = event.getX();
	                          mPendingCheckForTap.y = event.getY();
	                          postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
	                      } else {
	                          // Not inside a scrolling container, so show the feedback right away
	                          setPressed(true, x, y);
	                          checkForLongClick(
	                                  ViewConfiguration.getLongPressTimeout(),
	                                  x,
	                                  y,
	                                  TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__LONG_PRESS);
	                      }
	                      break;
	  
	                  case MotionEvent.ACTION_CANCEL:
	                      if (clickable) {
	                          setPressed(false);
	                      }
	                      removeTapCallback();
	                      removeLongPressCallback();
	                      mInContextButtonPress = false;
	                      mHasPerformedLongPress = false;
	                      mIgnoreNextUpEvent = false;
	                      mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
	                      break;
	  
	                  case MotionEvent.ACTION_MOVE:
	                      if (clickable) {
	                          drawableHotspotChanged(x, y);
	                      }
	  
	                      final int motionClassification = event.getClassification();
	                      final boolean ambiguousGesture =
	                              motionClassification == MotionEvent.CLASSIFICATION_AMBIGUOUS_GESTURE;
	                      int touchSlop = mTouchSlop;
	                      if (ambiguousGesture && hasPendingLongPressCallback()) {
	                          final float ambiguousMultiplier =
	                                  ViewConfiguration.getAmbiguousGestureMultiplier();
	                          if (!pointInView(x, y, touchSlop)) {
	                              // The default action here is to cancel long press. But instead, we
	                              // just extend the timeout here, in case the classification
	                              // stays ambiguous.
	                              removeLongPressCallback();
	                              long delay = (long) (ViewConfiguration.getLongPressTimeout()
	                                      * ambiguousMultiplier);
	                              // Subtract the time already spent
	                              delay -= event.getEventTime() - event.getDownTime();
	                              checkForLongClick(
	                                      delay,
	                                      x,
	                                      y,
	                                      TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__LONG_PRESS);
	                          }
	                          touchSlop *= ambiguousMultiplier;
	                      }
	  
	                      // Be lenient about moving outside of buttons
	                      if (!pointInView(x, y, touchSlop)) {
	                          // Outside button
	                          // Remove any future long press/tap checks
	                          removeTapCallback();
	                          removeLongPressCallback();
	                          if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
	                              setPressed(false);
	                          }
	                          mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
	                      }
	  
	                      final boolean deepPress =
	                              motionClassification == MotionEvent.CLASSIFICATION_DEEP_PRESS;
	                      if (deepPress && hasPendingLongPressCallback()) {
	                          // process the long click action immediately
	                          removeLongPressCallback();
	                          checkForLongClick(
	                                  0 /* send immediately */,
	                                  x,
	                                  y,
	                                  TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__DEEP_PRESS);
	                      }
	  
	                      break;
	              }
	  
	              return true;
	          }
	  
	          return false;
	      }
	  
	  ```
	-
- ## 在actionUp里处理了onClickListener
  collapsed:: true
	- ```java
	  if (mPerformClick == null) {
	    mPerformClick = new PerformClick();
	  }
	  if (!post(mPerformClick)) {
	    performClickInternal();
	  }
	  ```
	- performClickInternal
	  collapsed:: true
		- ```java
		      private boolean performClickInternal() {
		          // Must notify autofill manager before performing the click actions to avoid scenarios where
		          // the app has a click listener that changes the state of views the autofill service might
		          // be interested on.
		          notifyAutofillManagerOnClick();
		  
		          return performClick();
		      }
		  ```
	- performClick
		- ```java
		  public boolean performClick() {
		          // We still need to call this method to handle the cases where performClick() was called
		          // externally, instead of through performClickInternal()
		          notifyAutofillManagerOnClick();
		  
		          final boolean result;
		          final ListenerInfo li = mListenerInfo;
		          if (li != null && li.mOnClickListener != null) {
		              playSoundEffect(SoundEffectConstants.CLICK);
		              li.mOnClickListener.onClick(this);
		              result = true;
		          } else {
		              result = false;
		          }
		  
		          sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
		  
		          notifyEnterOrExitForAutoFillIfNeeded(true);
		  
		          return result;
		      }
		  ```
	- > 判断如果设置了mOnClickListener，则会走onClick方法。同时设置result = true 告诉父容器消费了
	  onClick只要走就会消费，onTouch是有返回值的，可以决定消费与否