title:: 属性动画 ValueAnimator.ofInt运行原理

	-
- # 前言
  collapsed:: true
	- ```
	  val valueAnimator = ValueAnimator.ofInt(0, 100,300).apply {
	              duration = 1000
	              interpolator = LinearInterpolator()
	              addUpdateListener {
	              }
	          }
	   valueAnimator.start()
	  ```
	- 问题：
	- 0-100 中经过的时间到底是0.5s还是0.33s？
	  ValueAnimator.ofInt(0, 300)有区别吗？
- # 源码解析
	- 那我们先从ValueAnimator.ofInt()开始分析
	  collapsed:: true
		- ```
		  //ValueAnimator
		  public static ValueAnimator ofInt(int... values) {
		          ValueAnimator anim = new ValueAnimator();
		          anim.setIntValues(values);
		          return anim;
		  }
		  ```
	- 我们重点是要查看values的去向,进入anim.setFloatValues(value)方法
	  collapsed:: true
		- ```
		  //ValueAnimator
		  public void setIntValues(int... values) {
		     if (values == null || values.length == 0) {
		          return;
		      }
		      if (mValues == null || mValues.length == 0) {
		          setValues(PropertyValuesHolder.ofInt("", values));
		      } else {
		          PropertyValuesHolder valuesHolder = mValues[0];
		          valuesHolder.setIntValues(values);
		      }
		      // New property/values/target should cause re-initialization prior to starting
		      mInitialized = false;
		  } 
		  ```
	- mValues为null，执行setValues(PropertyValuesHolder.ofInt("", values));最终调用了IntPropertyValuesHolder的构造方法
	  collapsed:: true
		- ```
		  //IntPropertyValuesHolder
		  public IntPropertyValuesHolder(String propertyName, int... values) {
		         super(propertyName);
		         setIntValues(values);
		  }
		  ```
		- ```
		  //IntPropertyValuesHolder
		  public void setIntValues(int... values) {
		        mValueType = int.class;
		        mKeyframes = KeyframeSet.ofInt(values);
		    }
		  ```
	- 这里会生成KeyframeSet
	  collapsed:: true
		- ```
		  //KeyframeSet
		  public static KeyframeSet ofInt(int... values) {
		     int numKeyframes = values.length;
		     IntKeyframe keyframes[] = new IntKeyframe[Math.max(numKeyframes,2)];
		     if (numKeyframes == 1) {
		         keyframes[0] = (IntKeyframe) Keyframe.ofInt(0f);
		         keyframes[1] = (IntKeyframe) Keyframe.ofInt(1f, values[0]);
		     } else {
		         keyframes[0] = (IntKeyframe) Keyframe.ofInt(0f, values[0]);
		         for (int i = 1; i < numKeyframes; ++i) {
		             keyframes[i] =
		                     (IntKeyframe) Keyframe.ofInt((float) i / (numKeyframes - 1), values[i]);
		         }
		     }
		     return new IntKeyframeSet(keyframes);
		  }
		  ```
	- 这里说明了我们在可变参数传一个的时候为什么动画是从0开始，原来在内部在初始化的时候进行了处理,初始化操作就到这里
	  collapsed:: true
		- ```
		  public static Keyframe ofInt(float fraction, int value) {
		     return new IntKeyframe(fraction, value);
		  }
		  
		  /**
		  * Internal subclass used when the keyframe value is of type int.
		  */
		  static class IntKeyframe extends Keyframe {
		  
		     /**
		      * The value of the animation at the time mFraction.
		      */
		     int mValue;
		  
		     IntKeyframe(float fraction, int value) {
		         mFraction = fraction;
		         mValue = value;
		         mValueType = int.class;
		         mHasValue = true;
		     }
		  }
		  ```
	- 在创建每个关键帧时，传入了两个参数，第一个参数就是表示这个关键帧在整个区域之间的位置，第二参数就是它表示的值是多少。看上面的代码， i 表示的是第几帧，numKeyframes 表示的是关键帧的总数量，所以 i/(numKeyframes - 1) 也就是表示这一系列关键帧是按等比例来分配的。
	- 到这里对我们前面提出的问题，还是没有解决？
	  只知道
	  0是第一帧(fraction = 0)
	  100是第二帧(fraction = 1/2)
	  300是第三帧(fraction = 1 )
	- ```
	  //IntKeyframeSet 
	  public IntKeyframeSet(IntKeyframe... keyframes) {
	        super(keyframes);
	   }
	  ```
	- ```java
	  public class KeyframeSet implements Keyframes {
	  
	  int mNumKeyframes;
	  
	  Keyframe mFirstKeyframe;
	  Keyframe mLastKeyframe;
	  TimeInterpolator mInterpolator; // only used in the 2-keyframe case
	  List<Keyframe> mKeyframes; // only used when there are not 2 keyframes
	  TypeEvaluator mEvaluator;
	  
	  public KeyframeSet(Keyframe... keyframes) {
	      mNumKeyframes = keyframes.length;
	      // immutable list
	      mKeyframes = Arrays.asList(keyframes);
	      mFirstKeyframe = keyframes[0];
	      mLastKeyframe = keyframes[mNumKeyframes - 1];
	      mInterpolator = mLastKeyframe.getInterpolator();
	  }
	  ```
	- 总结一下
	  collapsed:: true
		- ValueAnimator.ofInt时序图
			- ![image.png](../assets/image_1684419614171_0.png)
	- 现在ValueAnimator#mValues持有PropertyValuesHolder
	  PropertyValuesHolder#mKeyframes持有IntKeyframeSet
	  IntKeyframeSet#mKeyframes持有IntKeyframe
	  属性动画中什么时候用到了PropertyValuesHolder#mValues
	  那现在我们从入手valueAnimator.start()，看看什么时候用到了mValues
	- ## ValueAnimator.start()源码解析
		- 只能从animator.start()中去寻找答案了
		  collapsed:: true
		  ** ps:本次分析的是ValueAnimator，不是ObjectAnimator**
			- ```
			  @Override
			      public void start() {
			          start(false);
			      }
			      
			  	private void start(boolean playBackwards) {
			          if (Looper.myLooper() == null) {
			              throw new AndroidRuntimeException("Animators may only be run on Looper threads");
			          }
			          mReversing = playBackwards;
			          mSelfPulse = !mSuppressSelfPulseRequested;
			          // Special case: reversing from seek-to-0 should act as if not seeked at all.
			          if (playBackwards && mSeekFraction != -1 && mSeekFraction != 0) {
			              if (mRepeatCount == INFINITE) {
			                  // Calculate the fraction of the current iteration.
			                  float fraction = (float) (mSeekFraction - Math.floor(mSeekFraction));
			                  mSeekFraction = 1 - fraction;
			              } else {
			                  mSeekFraction = 1 + mRepeatCount - mSeekFraction;
			              }
			          }
			          mStarted = true;
			          mPaused = false;
			          mRunning = false;
			          mAnimationEndRequested = false;
			          // Resets mLastFrameTime when start() is called, so that if the animation was running,
			          // calling start() would put the animation in the
			          // started-but-not-yet-reached-the-first-frame phase.
			          mLastFrameTime = -1;
			          mFirstFrameTime = -1;
			          mStartTime = -1;
			          addAnimationCallback(0);
			  
			          if (mStartDelay == 0 || mSeekFraction >= 0 || mReversing) {
			              // If there's no start delay, init the animation and notify start listeners right away
			              // to be consistent with the previous behavior. Otherwise, postpone this until the first
			              // frame after the start delay.
			              startAnimation();
			              if (mSeekFraction == -1) {
			                  // No seek, start at play time 0. Note that the reason we are not using fraction 0
			                  // is because for animations with 0 duration, we want to be consistent with pre-N
			                  // behavior: skip to the final value immediately.
			                  setCurrentPlayTime(0);
			              } else {
			                  setCurrentFraction(mSeekFraction);
			              }
			          }
			      }
			  ```
		- 调用了内部的 start(boolean) 方法，前面无外乎就是一些变量的初始化，然后好像调用了很多方法，我们知道AnimationHandler是统一处理属性动画的，那么就找和AnimationHandler相关的逻辑。
		- ok找到了，在addAnimationCallback(0)中，我们找到了和AnimationHandler相关的逻辑，来看看：
			- ```
			  private void addAnimationCallback(long delay) {
			         if (!mSelfPulse) {
			             return;
			         }
			         getAnimationHandler().addAnimationFrameCallback(this, delay);
			  }
			  ```
			- ```
			   public void addAnimationFrameCallback(final AnimationFrameCallback callback, long delay) {
			   		/**
			   		* 重点
			   		*/
			          if (mAnimationCallbacks.size() == 0) {
			              getProvider().postFrameCallback(mFrameCallback);
			          }
			          if (!mAnimationCallbacks.contains(callback)) {
			              mAnimationCallbacks.add(callback);
			          }
			  
			          if (delay > 0) {
			              mDelayedCallbackStartTime.put(callback, (SystemClock.uptimeMillis() + delay));
			          }
			      }
			  ```
			- ```
			  /ValueAnimator
			  private AnimationFrameCallbackProvider getProvider() {
			          if (mProvider == null) {
			              mProvider = new MyFrameCallbackProvider();
			          }
			          return mProvider;
			   }
			  ```
			- ```
			  //MyFrameCallbackProvider
			  public void postFrameCallback(Choreographer.FrameCallback callback) {
			              mChoreographer.postFrameCallback(callback);
			   }
			  ```
			- 看到 Choreographer 了，感觉我们好像已经快接近真相了，不继续跟下去了。
			  大概说一下后面的流程：
			- 向底层注册屏幕监听信号：
			  Choreographer 内部有几个队列，通过postCallbackDelayedInternal(int callbackType,
			  Object action, Object token, long delayMillis) 方法将FrameCallback放进队列中，callbackType是区分队列类型的，属性动画中传入的是CALLBACK_ANIMATION用于区分这些队列。接着，Choreographer#scheduleFrameLocked()->scheduleFrameLocked#()scheduleVsyncLocked()->FrameDisplayEventReceiver#scheduleVsyncLocked()->最终调用native的方法nativeScheduleVsync（）
			  向底层注册屏幕监听信号。
			- 当屏幕刷新信号来临时：
			  nativeScheduleVsync()会向SurfaceFlinger注册Vsync信号的监听，VSync信号由SurfaceFlinger实现并定时发送，当Vsync信号来的时候就会回调FrameDisplayEventReceiver#onVsync()，这个方法给发送一个带时间戳Runnable消息，这个Runnable消息的run()实现就是FrameDisplayEventReceiver# run()， 接着就会执行doFrame()
			  具体的流程比如：nativeScheduleVsync怎么向SurfaceFlinger注册监听信号，以及SurfaceFlinger和VSync是什么？以后再去研究。
			- 根据目前的信息，我们先小结一下：
			  当 ValueAnimator 调用了 start() 方法之后，首先会对一些变量进行初始化工作并通知动画开始了，然后 ValueAnimator 实现了 AnimationFrameCallback 接口，并通过 AnimationHander 将自身 this 作为参数传到 mAnimationCallbacks 列表里缓存起来。而 AnimationHandler 在 mAnimationCallbacks 列表大小为 0 时会通过内部类 MyFrameCallbackProvider 将一个 mFrameCallback 工作缓存到 Choreographer 的待执行队列里，并向底层注册监听下一个屏幕刷新信号事件。
			  当屏幕刷新信号到的时候，Choreographer 的 doFrame() 会去将这些待执行队列里的工作取出来执行，那么此时也就回调了 AnimationHandler 的 mFrameCallback 工作。
			- 那么，接下去就继续看看，当接收到屏幕刷新信号之后，mFrameCallback 又继续做了什么：
				- ```
				  private final Choreographer.FrameCallback mFrameCallback = new Choreographer.FrameCallback() {
				          @Override
				          public void doFrame(long frameTimeNanos) {
				              doAnimationFrame(getProvider().getFrameTime());
				              if (mAnimationCallbacks.size() > 0) {
				                  getProvider().postFrameCallback(this);
				              }
				          }
				      };
				  ```
				- 1、去处理动画的相关工作，也就是说要找到动画真正执行的地方，
				- 2、继续向底层注册监听下一个屏幕刷新信号。
			- 那么，下去就是跟着 doAnimationFrame() 来看看，属性动画是怎么执行的
			  collapsed:: true
				- ```
				  //AnimationHandler
				   private void doAnimationFrame(long frameTime) {
				          long currentTime = SystemClock.uptimeMillis();
				          final int size = mAnimationCallbacks.size();
				          for (int i = 0; i < size; i++) {
				              final AnimationFrameCallback callback = mAnimationCallbacks.get(i);
				              if (callback == null) {
				                  continue;
				              }
				              if (isCallbackDue(callback, currentTime)) {
				                  callback.doAnimationFrame(frameTime);
				                  if (mCommitCallbacks.contains(callback)) {
				                      getProvider().postCommitCallback(new Runnable() {
				                          @Override
				                          public void run() {
				                              commitAnimationFrame(callback, getProvider().getFrameTime());
				                          }
				                      });
				                  }
				              }
				          }
				          cleanUpList();
				      }
				  ```
			- 1、是去循环遍历列表，取出每一个 ValueAnimator，就会去调用 ValueAnimator 的 doAnimationFrame()；
			- 2、是调用了 cleanUpList() 方法，处理掉已经结束的动画；
			  collapsed:: true
				- ```
				  //ValueAnimator
				  public final boolean doAnimationFrame(long frameTime) {
				       if (mStartTime < 0) {
				           // First frame. If there is start delay, start delay count down will happen *after* this
				           // frame.
				           mStartTime = mReversing
				                   ? frameTime
				                   : frameTime + (long) (mStartDelay * resolveDurationScale());
				       }
				  
				       // Handle pause/resume
				       if (mPaused) {
				           mPauseTime = frameTime;
				           removeAnimationCallback();
				           return false;
				       } else if (mResumed) {
				           mResumed = false;
				           if (mPauseTime > 0) {
				               // Offset by the duration that the animation was paused
				               mStartTime += (frameTime - mPauseTime);
				           }
				       }
				  
				       if (!mRunning) {
				           // If not running, that means the animation is in the start delay phase of a forward
				           // running animation. In the case of reversing, we want to run start delay in the end.
				           if (mStartTime > frameTime && mSeekFraction == -1) {
				               // This is when no seek fraction is set during start delay. If developers change the
				               // seek fraction during the delay, animation will start from the seeked position
				               // right away.
				               return false;
				           } else {
				               // If mRunning is not set by now, that means non-zero start delay,
				               // no seeking, not reversing. At this point, start delay has passed.
				               mRunning = true;
				               startAnimation();
				           }
				       }
				  
				       if (mLastFrameTime < 0) {
				           if (mSeekFraction >= 0) {
				               long seekTime = (long) (getScaledDuration() * mSeekFraction);
				               mStartTime = frameTime - seekTime;
				               mSeekFraction = -1;
				           }
				           mStartTimeCommitted = false; // allow start time to be compensated for jank
				       }
				       mLastFrameTime = frameTime;
				       // The frame time might be before the start time during the first frame of
				       // an animation.  The "current time" must always be on or after the start
				       // time to avoid animating frames at negative time intervals.  In practice, this
				       // is very rare and only happens when seeking backwards.
				       final long currentTime = Math.max(frameTime, mStartTime);
				       boolean finished = animateBasedOnTime(currentTime);
				  
				       if (finished) {
				           endAnimation();
				       }
				       return finished;
				   }
				  ```
			- 稍微概括一下，这个方法内部其实就做了三件事：
			  collapsed:: true
				- 1 是处理第一帧动画的一些工作；
				- 2 是根据当前时间计算当前帧的动画进度，所以动画的核心应该就是在 animateBaseOnTime() 这个方法里（刚开始问题的答案，或许也在这里面;
				- 3 是判断动画是否已经结束了，结束了就去调用 endAnimation();
				  我们重点在第二件事中查找答案：
				- ```
				  //ValueAnimator
				  boolean animateBasedOnTime(long currentTime) {
				          boolean done = false;
				          if (mRunning) {
				              final long scaledDuration = getScaledDuration();
				              final float fraction = scaledDuration > 0 ?
				                      (float)(currentTime - mStartTime) / scaledDuration : 1f;
				              final float lastFraction = mOverallFraction;
				              final boolean newIteration = (int) fraction > (int) lastFraction;
				              final boolean lastIterationFinished = (fraction >= mRepeatCount + 1) &&
				                      (mRepeatCount != INFINITE);
				              if (scaledDuration == 0) {
				                  // 0 duration animator, ignore the repeat count and skip to the end
				                  done = true;
				              } else if (newIteration && !lastIterationFinished) {
				                  // Time to repeat
				                  if (mListeners != null) {
				                      int numListeners = mListeners.size();
				                      for (int i = 0; i < numListeners; ++i) {
				                          mListeners.get(i).onAnimationRepeat(this);
				                      }
				                  }
				              } else if (lastIterationFinished) {
				                  done = true;
				              }
				              mOverallFraction = clampFraction(fraction);
				              float currentIterationFraction = getCurrentIterationFraction(
				                      mOverallFraction, mReversing);
				              animateValue(currentIterationFraction);
				          }
				          return done;
				      }
				  ```
			- 根据当前时间以及动画第一帧时间还有动画持续的时长来计算当前的动画进度。
			  确保这个动画进度的取值在 0-1 之间，这里调用了两个方法来辅助计算，我们就不跟进去了，之所以有这么多的辅助计算，那是因为，属性动画支持 setRepeatCount() 来设置动画的循环次数，而从始至终的动画第一帧的时间都是 mStrtTime 一个值，所以在第一个步骤中根据当前时间计算动画进度时会发现进度值是可能会超过 1 的，比如 1.5, 2.5, 3.5 等等，所以第二个步骤的辅助计算，就是将这些值等价换算到 0-1 之间。
			- 我们首先fraction 是怎么计算的
			  collapsed:: true
				- ```
				  final float fraction = scaledDuration > 0 ?
				                      (float)(currentTime - mStartTime) / scaledDuration : 1f;
				  ```
			- 每个动画在处理当前帧的动画逻辑时，首先会先根据当前时间和动画第一帧时间以及动画的持续时长来初步计算出当前帧时动画所处的进度。
			- ```
			  //ValueAnimator
			  void animateValue(float fraction) {
			          fraction = mInterpolator.getInterpolation(fraction);
			          mCurrentFraction = fraction;
			          int numValues = mValues.length;
			          for (int i = 0; i < numValues; ++i) {
			              mValues[i].calculateValue(fraction);
			          }
			          if (mUpdateListeners != null) {
			              int numListeners = mUpdateListeners.size();
			              for (int i = 0; i < numListeners; ++i) {
			                  mUpdateListeners.get(i).onAnimationUpdate(this);
			              }
			          }
			      }
			  ```
			- mUpdateListeners.get(i).onAnimationUpdate(this); 这是不是我们刚开始给ValueAnimator设置的addUpdateListener。参数也是ValueAnimator,也就是在这个函数中，计算了当前属性动画所属于的区间，和经过的时长，然后通知动画的进度回调
			- 终于找到了mValues是在这里使用计算的
			  这部分工作主要就是调用了 mValues[i].calculateValue(fraction) 这一行代码来实现，mValues 是一个 PropertyValuesHolder 类型的数组，所以关键就是去看看这个类的 calculateValue() 做了啥：
	-