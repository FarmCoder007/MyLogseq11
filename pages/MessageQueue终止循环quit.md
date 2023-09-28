### 源码
	- ```java
	   void quit(boolean safe) {
	          if (!mQuitAllowed) {
	              throw new IllegalStateException("Main thread not allowed to quit.");
	          }
	  
	          synchronized (this) {
	              if (mQuitting) {
	                  return;
	              }
	              // 1、设置标记可以让 next()方法返回null，终止loop循环
	              mQuitting = true;
	   			// 2、移除所有消息
	              if (safe) {
	                  removeAllFutureMessagesLocked();
	              } else {
	                  removeAllMessagesLocked();
	              }
	  			// 3、唤醒线程
	              // We can assume mPtr != 0 because mQuitting was previously false.
	              nativeWake(mPtr);
	          }
	      }
	  ```
- ### 做的事情：
	- 1、设置标记mQuitting = true
		- （标记作用后期循环醒了可以让 next()方法返回null，终止loop循环）
	- 2、移除所有消息【释放内存】
	- 3、唤醒当前线程nativeWake(mPtr);    通知messageQueue里的next()里的  nativePollOnce  唤醒  继续取数据，判断mQuitting：true,next（） 返回msg为null。从而终止Loop()
		- Looper.java的 loop函数
			- ```java
			      for (;;) {
			              Message msg = queue.next(); // might block
			              if (msg == null) {
			                  // No message indicates that the message queue is quitting.
			                  return;
			              }
			              。。。。。。
			           }
			  ```