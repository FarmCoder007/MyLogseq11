title:: Handler().runWithScissors 同步等待

- Handler().runWithScissors 同步等待，等待handler.post或者 sendMessage代码执行后，再继续向下执行，达到同步效果。
- ## 源代码
	- ```java
	      public final boolean runWithScissors(@NonNull Runnable r, long timeout) {
	          if (r == null) {
	              throw new IllegalArgumentException("runnable must not be null");
	          }
	          if (timeout < 0) {
	              throw new IllegalArgumentException("timeout must be non-negative");
	          }
	  
	          // 1、如果是在 当前线程执行，那么执行run
	          if (Looper.myLooper() == mLooper) {
	              r.run();
	              return true;
	          }
	          // 2、否则将runnable 转换成  BlockingRunnable, 然后执行postAndWait
	          BlockingRunnable br = new BlockingRunnable(r);
	          return br.postAndWait(this, timeout);
	      }
	  
	  	       
	  ```
	- ```java
	  // 执行任务 并阻塞，直到任务执行完 
	  public boolean postAndWait(Handler handler, long timeout) {
	              if (!handler.post(this)) {
	                  return false;
	              }
	  
	              synchronized (this) {
	                  if (timeout > 0) {
	                      final long expirationTime = SystemClock.uptimeMillis() + timeout;
	                      while (!mDone) {
	                          long delay = expirationTime - SystemClock.uptimeMillis();
	                          if (delay <= 0) {
	                              return false; // timeout
	                          }
	                          try {
	                              wait(delay);
	                          } catch (InterruptedException ex) {
	                          }
	                      }
	                  } else {
	                      while (!mDone) {
	                          try {
	                              wait();
	                          } catch (InterruptedException ex) {
	                          }
	                      }
	                  }
	              }
	              return true;
	          }
	  ```