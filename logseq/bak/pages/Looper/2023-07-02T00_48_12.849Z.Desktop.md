- # 核心1：构造函数
  collapsed:: true
	- ## 1、私有构造函数,只能通过prepare初始化
		- ```java
		      private Looper(boolean quitAllowed) {
		          mQueue = new MessageQueue(quitAllowed);
		          mThread = Thread.currentThread();
		      }
		  ```
	- ## 2、prepare
		- ```java
		      private static void prepare(boolean quitAllowed) {
		          if (sThreadLocal.get() != null) {
		              throw new RuntimeException("Only one Looper may be created per thread");
		          }
		          sThreadLocal.set(new Looper(quitAllowed));
		      }
		  ```
- # 核心2：loop函数
	- Looper.loop()启动1个死循环
	- ### 怎么退出死循环？
	  collapsed:: true
		- 源码：出现msg = null的时候就会退出
		- ```java
		  if (msg == null) {
		                  // No message indicates that the message queue is quitting.
		                  return;
		              }
		  
		  ```
	- ### 什么时候msg = null？
	  collapsed:: true
		- 1、程序退出调用quit，调用loop.quit
	- ### 源码
		- ```java
		   /**
		       * Run the message queue in this thread. Be sure to call
		       * {@link #quit()} to end the loop.
		       */
		      public static void loop() {
		          final Looper me = myLooper();
		          if (me == null) {
		              throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
		          }
		          final MessageQueue queue = me.mQueue;
		  
		          // Make sure the identity of this thread is that of the local process,
		          // and keep track of what that identity token actually is.
		          Binder.clearCallingIdentity();
		          final long ident = Binder.clearCallingIdentity();
		  
		          // Allow overriding a threshold with a system prop. e.g.
		          // adb shell 'setprop log.looper.1000.main.slow 1 && stop && start'
		          final int thresholdOverride =
		                  SystemProperties.getInt("log.looper."
		                          + Process.myUid() + "."
		                          + Thread.currentThread().getName()
		                          + ".slow", 0);
		  
		          boolean slowDeliveryDetected = false;
		  
		          for (;;) {
		              Message msg = queue.next(); // might block
		              if (msg == null) {
		                  // No message indicates that the message queue is quitting.
		                  return;
		              }
		  
		              // This must be in a local variable, in case a UI event sets the logger
		              final Printer logging = me.mLogging;
		              if (logging != null) {
		                  logging.println(">>>>> Dispatching to " + msg.target + " " +
		                          msg.callback + ": " + msg.what);
		              }
		              // Make sure the observer won't change while processing a transaction.
		              final Observer observer = sObserver;
		  
		              final long traceTag = me.mTraceTag;
		              long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;
		              long slowDeliveryThresholdMs = me.mSlowDeliveryThresholdMs;
		              if (thresholdOverride > 0) {
		                  slowDispatchThresholdMs = thresholdOverride;
		                  slowDeliveryThresholdMs = thresholdOverride;
		              }
		              final boolean logSlowDelivery = (slowDeliveryThresholdMs > 0) && (msg.when > 0);
		              final boolean logSlowDispatch = (slowDispatchThresholdMs > 0);
		  
		              final boolean needStartTime = logSlowDelivery || logSlowDispatch;
		              final boolean needEndTime = logSlowDispatch;
		  
		              if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
		                  Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
		              }
		  
		              final long dispatchStart = needStartTime ? SystemClock.uptimeMillis() : 0;
		              final long dispatchEnd;
		              Object token = null;
		              if (observer != null) {
		                  token = observer.messageDispatchStarting();
		              }
		              long origWorkSource = ThreadLocalWorkSource.setUid(msg.workSourceUid);
		              try {
		                  msg.target.dispatchMessage(msg);
		                  if (observer != null) {
		                      observer.messageDispatched(token, msg);
		                  }
		                  dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
		              } catch (Exception exception) {
		                  if (observer != null) {
		                      observer.dispatchingThrewException(token, msg, exception);
		                  }
		                  throw exception;
		              } finally {
		                  ThreadLocalWorkSource.restore(origWorkSource);
		                  if (traceTag != 0) {
		                      Trace.traceEnd(traceTag);
		                  }
		              }
		              if (logSlowDelivery) {
		                  if (slowDeliveryDetected) {
		                      if ((dispatchStart - msg.when) <= 10) {
		                          Slog.w(TAG, "Drained");
		                          slowDeliveryDetected = false;
		                      }
		                  } else {
		                      if (showSlowLog(slowDeliveryThresholdMs, msg.when, dispatchStart, "delivery",
		                              msg)) {
		                          // Once we write a slow delivery log, suppress until the queue drains.
		                          slowDeliveryDetected = true;
		                      }
		                  }
		              }
		              if (logSlowDispatch) {
		                  showSlowLog(slowDispatchThresholdMs, dispatchStart, dispatchEnd, "dispatch", msg);
		              }
		  
		              if (logging != null) {
		                  logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
		              }
		  
		              // Make sure that during the course of dispatching the
		              // identity of the thread wasn't corrupted.
		              final long newIdent = Binder.clearCallingIdentity();
		              if (ident != newIdent) {
		                  Log.wtf(TAG, "Thread identity changed from 0x"
		                          + Long.toHexString(ident) + " to 0x"
		                          + Long.toHexString(newIdent) + " while dispatching to "
		                          + msg.target.getClass().getName() + " "
		                          + msg.callback + " what=" + msg.what);
		              }
		  
		              msg.recycleUnchecked();
		          }
		      }
		  ```
- # 核心3：[[ThreadLocal]]函数
  collapsed:: true
	- ### [[Handler中的如何保证一个线程只有一个Looper]]
- # 核心4：quit，停止循环清空队列，释放内存
  collapsed:: true
	- 1、由Loop（）函数知道：启动了死循环，只有msg = null的时候才能退出
	- 2、msg是从消息队列里取出的，看消息队列next（）什么时候返回null
		- [[MessageQueue.next()]]中变量mQuitting 为true的时候
	- 3、该变量是队列调用quit时候 赋值true的,同时清空队列
	  collapsed:: true
		- ```java
		      void quit(boolean safe) {
		          if (!mQuitAllowed) {
		              throw new IllegalStateException("Main thread not allowed to quit.");
		          }
		  
		          synchronized (this) {
		              if (mQuitting) {
		                  return;
		              }
		              // 赋值变量是的 next()可以返回null
		              mQuitting = true;
		  
		              // 清空消息队列
		              if (safe) {
		                  removeAllFutureMessagesLocked();
		              } else {
		                  removeAllMessagesLocked();
		              }
		  
		              // We can assume mPtr != 0 because mQuitting was previously false.
		              // 唤醒
		              nativeWake(mPtr);
		          }
		      }
		  ```
	- 4、  mQueue.quit 由Looper.quit / Looper.quitSafely调用
	  collapsed:: true
		- ```java
		      /**
		       * Quits the looper safely.
		       * <p>
		       * Causes the {@link #loop} method to terminate as soon as all remaining messages
		       * in the message queue that are already due to be delivered have been handled.
		       * However pending delayed messages with due times in the future will not be
		       * delivered before the loop terminates.
		       * </p><p>
		       * Any attempt to post messages to the queue after the looper is asked to quit will fail.
		       * For example, the {@link Handler#sendMessage(Message)} method will return false.
		       * </p>
		       */
		      public void quitSafely() {
		          mQueue.quit(true);
		      }
		  ```
	- ### quit做的几件事
		- 1、会调用到 [[MessageQueue终止循环quit]]具体事情在里边
-