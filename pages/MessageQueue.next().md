title:: MessageQueue.next()

- messageQueue.next()
  collapsed:: true
	- ```java
	  Message next() {
	          // Return here if the message loop has already quit and been disposed.
	          // This can happen if the application tries to restart a looper after quit
	          // which is not supported.
	          final long ptr = mPtr;
	          if (ptr == 0) {
	              return null;
	          }
	  
	          int pendingIdleHandlerCount = -1; // -1 only during first iteration
	    // 1.如果nextPollTimeoutMillis=-1，一直阻塞不会超时。
	  // 2.如果nextPollTimeoutMillis=0，不会阻塞，立即返回。
	  // 3.如果nextPollTimeoutMillis>0，最长阻塞nextPollTimeoutMillis毫秒(超时)
	  // 如果期间有程序唤醒会立即返回
	          int nextPollTimeoutMillis = 0;
	    //next()也是一个无限循环
	          for (;;) {
	              if (nextPollTimeoutMillis != 0) {
	                  Binder.flushPendingCommands();
	              }
	  
	              nativePollOnce(ptr, nextPollTimeoutMillis); // 睡眠
	  
	              synchronized (this) {
	                  // Try to retrieve the next message.  Return if found.
	                //获取系统开机到现在的时间
	                  final long now = SystemClock.uptimeMillis();
	                  Message prevMsg = null;
	                  Message msg = mMessages;//当前链表的头结点
	                  ////如果target==null，那么它就是屏障，需要循环遍历，一直往后找到第一个异步的消息
	                  if (msg != null && msg.target == null) {
	                      // Stalled by a barrier.  Find the next asynchronous message in the queue.
	                      do {
	                          prevMsg = msg;
	                          msg = msg.next;
	                      } while (msg != null && !msg.isAsynchronous());
	                  }
	                  if (msg != null) {
	                    // 如果有消息需要处理，先判断时间有没有到，如果没到的话设置一下阻塞时间，
	                    //场景如常用的postDelay
	                      if (now < msg.when) { // 取出来的消息 未到执行时间，进入下次循环的时候会睡眠
	                          // Next message is not ready.  Set a timeout to wake up when it is ready.
	                         //计算出离执行时间还有多久赋值给nextPollTimeoutMillis，
	  						//表示nativePollOnce方法要等待nextPollTimeoutMillis时长后返回 
	                         nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
	                      } else {
	                          // Got a message
	                        // 获取到消息.
	                          mBlocked = false;
	                        //链表操作，获取msg并且删除该节点
	                          if (prevMsg != null) {
	                              prevMsg.next = msg.next;
	                          } else {
	                              mMessages = msg.next;
	                          }
	                          msg.next = null;
	                          if (DEBUG) Log.v(TAG, "Returning message: " + msg);
	                          msg.markInUse();
	                        //返回拿到的消息
	                          return msg;
	                      }
	                  } else {
	                      //没有消息，nextPollTimeoutMillis复位
	                      nextPollTimeoutMillis = -1;
	                  }
	  
	                  // Process the quit message now that all pending messages have been handled.
	                  // 终止的话 返回msg null
	                  if (mQuitting) {
	                      dispose();
	                      return null;
	                  }
	  
	                  // If first time idle, then get the number of idlers to run.
	                  // Idle handles only run if the queue is empty or if the first message
	                  // in the queue (possibly a barrier) is due to be handled in the future.
	                  if (pendingIdleHandlerCount < 0
	                          && (mMessages == null || now < mMessages.when)) {
	                      pendingIdleHandlerCount = mIdleHandlers.size();
	                  }
	                  if (pendingIdleHandlerCount <= 0) {
	                      // No idle handlers to run.  Loop and wait some more.
	                      mBlocked = true;
	                      continue;
	                  }
	  
	                  if (mPendingIdleHandlers == null) {
	                      mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
	                  }
	                  mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
	              }
	  
	              // Run the idle handlers.
	              // We only ever reach this code block during the first iteration.
	              for (int i = 0; i < pendingIdleHandlerCount; i++) {
	                  final IdleHandler idler = mPendingIdleHandlers[i];
	                  mPendingIdleHandlers[i] = null; // release the reference to the handler
	  
	                  boolean keep = false;
	                  try {
	                      keep = idler.queueIdle();
	                  } catch (Throwable t) {
	                      Log.wtf(TAG, "IdleHandler threw exception", t);
	                  }
	  
	                  if (!keep) {
	                      synchronized (this) {
	                          mIdleHandlers.remove(idler);
	                      }
	                  }
	              }
	  
	              // Reset the idle handler count to 0 so we do not run them again.
	              pendingIdleHandlerCount = 0;
	  
	              // While calling an idle handler, a new message could have been delivered
	              // so go back and look again for a pending message without waiting.
	              nextPollTimeoutMillis = 0;
	          }
	      }
	  ```
- 只有成员变量mQuitting = true的时候才会返回null
	- ```java
	       if (mQuitting) {
	                      dispose();
	                      return null;
	                  }
	  ```