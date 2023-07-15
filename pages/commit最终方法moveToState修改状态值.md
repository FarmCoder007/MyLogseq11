- ## FragmentManager 对这些方法的实现也很简单，修改 Fragment 的状态值，比如remove(Fragment) :
  collapsed:: true
	- removeFragment
		- ```java
		  public void removeFragment(Fragment fragment) {
		      if (DEBUG) Log.v(TAG, "remove: " + fragment + " nesting=" + 
		                       						fragment.mBackStackNesting);
		      final boolean inactive = !fragment.isInBackStack();
		      if (!fragment.mDetached || inactive) {
		          if (mAdded != null) {
		              mAdded.remove(fragment);
		          }
		          if (fragment.mHasMenu && fragment.mMenuVisible) {
		              mNeedMenuInvalidate = true;
		          }
		          fragment.mAdded = false; //设置属性值
		          fragment.mRemoving = true;
		      }
		  }
		  ```
- ## 最终会调用 moveToState() ，我们直接来看它的实现：(Fragment.java)
  collapsed:: true
	- ```java
	  void moveToState(Fragment f, int newState, int transit, int transitionStyle,
	  														boolean keepActive) {
	      //还没有添加的 Fragment 处于 onCreate() 状态
	      if ((!f.mAdded || f.mDetached) && newState > Fragment.CREATED) {
	          newState = Fragment.CREATED;
	      }
	      if (f.mRemoving && newState > f.mState) {
	          // While removing a fragment, we can't change it to a higher state.
	          newState = f.mState;
	      }
	      //推迟启动的设置为 stop
	      if (f.mDeferStart && f.mState < Fragment.STARTED && newState > Fragment.STOPPED) {
	          newState = Fragment.STOPPED;
	      }
	      if (f.mState < newState) {
	      // For fragments that are created from a layout, when restoring from
	      // state we don't want to allow them to be created until they are
	      // being reloaded from the layout.
	      if (f.mFromLayout && !f.mInLayout) {
	          return;
	      }
	      if (f.getAnimatingAway() != null) {
	          // The fragment is currently being animated... but! Now we
	          // want to move our state back up. Give up on waiting for the
	          // animation, move to whatever the final state should be once
	          // the animation is done, and then we can proceed from there.
	          f.setAnimatingAway(null);
	          //如果当前 Fragment 正有动画，直接修改为最终状态
	          moveToState(f, f.getStateAfterAnimating(), 0, 0, true);
	      }
	  ```
	- ```java
	  
	  switch (f.mState) {
	      case Fragment.INITIALIZING:
	      	if (DEBUG) Log.v(TAG, "moveto CREATED: " + f);
	              if (f.mSavedFragmentState != null) {
	                  f.mSavedFragmentState.setClassLoader(mHost.getContext().getClassLoader());
	                  f.mSavedViewState =
	                              f.mSavedFragmentState.getSparseParcelableArray(
	                                                      FragmentManagerImpl.VIEW_STATE_TAG);
	                                                  f.mTarget = getFragment(f.mSavedFragmentState,
	                                                      FragmentManagerImpl.TARGET_STATE_TAG);
	                  if (f.mTarget != null) {
	                      f.mTargetRequestCode =
	                      f.mSavedFragmentState.getInt(FragmentManagerImpl.
	                                                            TARGET_REQUEST_CODE_STATE_TAG, 0);
	                  }
	                  f.mUserVisibleHint = f.mSavedFragmentState.getBoolean(
	                              FragmentManagerImpl.USER_VISIBLE_HINT_TAG,true);
	                  if (!f.mUserVisibleHint) {
	                      f.mDeferStart = true;
	                      if (newState > Fragment.STOPPED) {
	                          newState = Fragment.STOPPED;
	                      }
	                  }
	              }
	              f.mHost = mHost;
	              f.mParentFragment = mParent;
	              f.mFragmentManager = mParent != null :mParent.mChildFragmentManager :
	                                                              mHost.getFragmentManagerImpl();
	              dispatchOnFragmentPreAttached(f, mHost.getContext(),false);
	              f.mCalled = false;
	              f.onAttach(mHost.getContext()); //调用 Fragment 生命周期方法
	              if (!f.mCalled) {
	                  throw new SuperNotCalledException("Fragment " + f+ " did not call through to
	                                                                      super.onAttach()");
	              }
	              if (f.mParentFragment == null) {
	                  mHost.onAttachFragment(f);
	              } else {
	                  f.mParentFragment.onAttachFragment(f);
	              }
	              dispatchOnFragmentAttached(f, mHost.getContext(), false);
	              if (!f.mRetaining) {
	                    //调用 Fragment生命周期方法
	                  f.performCreate(f.mSavedFragmentState); 
	                  dispatchOnFragmentCreated(f, f.mSavedFragmentState,false);
	              } else {
	                  f.restoreChildFragmentState(f.mSavedFragmentState);
	                  f.mState = Fragment.CREATED;
	              }
	              f.mRetaining = false;
	              if (f.mFromLayout) { //从布局解析来的
	                  // For fragments that are part of the content view
	                  // layout, we need to instantiate the view immediately
	                  // and the inflater will take care of adding it.
	                  f.mView = f.performCreateView(f.getLayoutInflater(
	                  //调用 Fragment 生命周期方法
	                  f.mSavedFragmentState), null,
	                  f.mSavedFragmentState);
	                  if (f.mView != null) {
	                      f.mInnerView = f.mView;
	                      if (Build.VERSION.SDK_INT >= 11) {
	                          ViewCompat.setSaveFromParentEnabled(f.mView,  false);
	                      } else {
	                          f.mView = NSaveStateFrameLayout.wrap(f.mView);
	                      }
	                      if (f.mHidden) f.mView.setVisibility(View.GONE);
	                          f.onViewCreated(f.mView, f.mSavedFragmentState);
	                          //调用 Fragment 生命周期方法
	                          dispatchOnFragmentViewCreated(f, f.mView,f.mSavedFragmentState, false);
	                      } else {
	                          f.mInnerView = null;
	                      }
	                  }
	      case Fragment.CREATED:
	      if (newState > Fragment.CREATED) {
	      	if (DEBUG) Log.v(TAG, "moveto ACTIVITY_CREATED: " +f);
	              if (!f.mFromLayout) {
	                  ViewGroup container = null;
	                  if (f.mContainerId != 0) {
	                      if (f.mContainerId == View.NO_ID) {
	                          throwException(new
	                          IllegalArgumentException(
	                          "Cannot create fragment "
	                          + f
	                          + " for a container view
	                          with no id"));
	                      }
	                      container = (ViewGroup)
	                      mContainer.onFindViewById(f.mContainerId);
	                      if (container == null && !f.mRestored) {
	                          String resName;
	                          try {
	                              resName =f.getResources().getResourceName(f.mContainerId);
	                          } catch (NotFoundException e) {
	                              resName = "unknown";
	                          }
	                          throwException(new
	                              IllegalArgumentException(
	                              "No view found for id 0x"
	                              +
	                              Integer.toHexString(f.mContainerId) + " ("
	                                + resName
	                              + ") for fragment " + f));
	                      }
	                  }
	                  f.mContainer = container;
	                  f.mView = f.performCreateView(f.getLayoutInflater(
	                  //调用 Fragment 生命周期方法
	                  f.mSavedFragmentState), container,
	                  f.mSavedFragmentState);
	                  if (f.mView != null) {
	                      f.mInnerView = f.mView;
	                      if (Build.VERSION.SDK_INT >= 11) {
	                          ViewCompat.setSaveFromParentEnabled(f.mView, false);
	                      } else {
	                          f.mView =NoSaveStateFrameLayout.wrap(f.mView);
	                      }
	                      if (container != null) {
	                         //将Fragment 的布局添加到父布局中
	                          container.addView(f.mView);
	                      }
	                      if (f.mHidden) {
	                          f.mView.setVisibility(View.GONE);
	                      }
	                      f.onViewCreated(f.mView,
	                      f.mSavedFragmentState);//调用 Fragment 生命周期方法
	                      dispatchOnFragmentViewCreated(f, f.mView,f.mSavedFragmentState,false);
	                      // Only animate the view if it is visible.This is done after
	                      // dispatchOnFragmentViewCreated in case isibility is changed
	                      f.mIsNewlyAdded = (f.mView.getVisibility() ==View.VISIBLE)&& f.mContainer != null;
	                  } else {
	                      f.mInnerView = null;
	                  }
	      		}
	            f.performActivityCreated(f.mSavedFragmentState); //调用Fragment 生命周期方法
	            dispatchOnFragmentActivityCreated(f,f.mSavedFragmentState, false);
	            if (f.mView != null) {
	                f.restoreViewState(f.mSavedFragmentState);
	            }
	            f.mSavedFragmentState = null;
	     	 }
	      case Fragment.ACTIVITY_CREATED:
	          if (newState > Fragment.ACTIVITY_CREATED) {
	              f.mState = Fragment.STOPPED;
	          }
	      case Fragment.STOPPED:
	          if (newState > Fragment.STOPPED) {
	              if (DEBUG) Log.v(TAG, "moveto STARTED: " + f);
	              f.performStart();
	              dispatchOnFragmentStarted(f, false);
	          }
	      case Fragment.STARTED:
	            if (newState > Fragment.STARTED) {
	                if (DEBUG) Log.v(TAG, "moveto RESUMED: " + f);
	                f.performResume();
	                dispatchOnFragmentResumed(f, false);
	                f.mSavedFragmentState = null;
	                f.mSavedViewState = null;
	            }
	  	}
	  } else if (f.mState > newState) {
	  ```
	- ```java
	  
	  switch (f.mState) {
	      case Fragment.RESUMED:
	          if (newState < Fragment.RESUMED) {
	              if (DEBUG) Log.v(TAG, "movefrom RESUMED: " + f);
	              f.performPause();
	              dispatchOnFragmentPaused(f, false);
	          }
	      case Fragment.STARTED:
	          if (newState < Fragment.STARTED) {
	              if (DEBUG) Log.v(TAG, "movefrom STARTED: " + f);
	              f.performStop();
	              dispatchOnFragmentStopped(f, false);
	          }
	      case Fragment.STOPPED:
	          if (newState < Fragment.STOPPED) {
	              if (DEBUG) Log.v(TAG, "movefrom STOPPED: " + f);
	              f.performReallyStop();
	          }
	  case Fragment.ACTIVITY_CREATED:
	      if (newState < Fragment.ACTIVITY_CREATED) {
	          if (DEBUG) Log.v(TAG, "movefrom ACTIVITY_CREATED: " +f);
	              if (f.mView != null) {
	                  // Need to save the current view state if not
	                  // done already.
	                  if (mHost.onShouldSaveFragmentState(f) &&f.mSavedViewState == null) {
	                      saveFragmentViewState(f);
	                  }
	              }
	              f.performDestroyView();
	              dispatchOnFragmentViewDestroyed(f, false);
	              if (f.mView != null && f.mContainer != null) {
	                  Animation anim = null;
	                  if (mCurState > Fragment.INITIALIZING &&
	                                      !mDestroyed
	                                      && f.mView.getVisibility() == View.VISIBLE
	                                      && f.mPostponedAlpha >= 0) {
	                      anim = loadAnimation(f, transit, false,
	                      transitionStyle);
	                  }
	                  f.mPostponedAlpha = 0;
	                  if (anim != null) {
	                      final Fragment fragment = f;
	                      f.setAnimatingAway(f.mView);
	                      f.setStateAfterAnimating(newState);
	                      final View viewToAnimate = f.mView;
	                      anim.setAnimationListener(new nimateOnHWLayerIfNeededListener(viewToAnimate, anim) {
	                            @Override
	                            public void onAnimationEnd(Animation nimation) {
	                                super.onAnimationEnd(animation);
	                                if (fragment.getAnimatingAway() !=null) {
	                                    fragment.setAnimatingAway(null);
	                                    moveToState(fragment,fragment.getStateAfterAnimating(),0, 0, false);
	                                }
	                            }
	                      });
	                      f.mView.startAnimation(anim);
	                  }
	                  f.mContainer.removeView(f.mView);
	              }
	              f.mContainer = null;
	              f.mView = null;
	              f.mInnerView = null;
	          }
	  case Fragment.CREATED:
	      if (newState < Fragment.CREATED) {
	          if (mDestroyed) {
	              if (f.getAnimatingAway() != null) {
	                  // The fragment's containing activity is
	                  // being destroyed, but this fragment is
	                  // currently animating away. Stop the
	                  // animation right now -- it is not needed,
	                  // and we can't wait any more on destroying
	                  // the fragment.
	                  View v = f.getAnimatingAway();
	                  f.setAnimatingAway(null);
	                  v.clearAnimation();
	              }
	          }
	          if (f.getAnimatingAway() != null) {
	              // We are waiting for the fragment's view to inish
	              // animating away. Just make a note of the state
	              // the fragment now should move to once the nimation
	              // is done.
	              f.setStateAfterAnimating(newState);
	              newState = Fragment.CREATED;
	           } else {
	                  if (DEBUG) Log.v(TAG, "movefrom CREATED: " + f);
	                      if (!f.mRetaining) {
	                        f.performDestroy();
	                        dispatchOnFragmentDestroyed(f, false);
	                      } else {
	                          f.mState = Fragment.INITIALIZING;
	                      }
	                      f.performDetach();
	                      dispatchOnFragmentDetached(f, false);
	                      if (!keepActive) {
	                          if (!f.mRetaining) {
	                              makeInactive(f);
	                          } else {
	                              f.mHost = null;
	                              f.mParentFragment = null;
	                              f.mFragmentManager = null;
	                          }
	                      }
	                  }
	              }
	          }
	      }
	      if (f.mState != newState) {
	          Log.w(TAG, "moveToState: Fragment state for " + f + " not updated
	          inline; "
	          + "expected state " + newState + " found " + f.mState);
	          f.mState = newState;
	      }
	  }
	  ```
- # 总结：代码很长，但做的事情很简单：
	- 1. 根据状态调用对应的生命周期方法
	- 2. 如果是新创建的，就把布局添加到 ViewGroup 中
-