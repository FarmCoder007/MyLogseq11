- 1、由 [[Looper]]中的loop()函数可知，从消息队列.next()取出消息后，交由Handler的dispatchMsg。进而到Handler的handMessage处理
- 2、处理完消息后会调用回收消息：msg.recycleUnchecked();并加入缓存池中。见[[设计模式]]享元模式
	- Message.java
	  collapsed:: true
		- ```java
		   void recycleUnchecked() {
		          // Mark the message as in use while it remains in the recycled object pool.
		          // Clear out all other details.
		          flags = FLAG_IN_USE;
		          what = 0;
		          arg1 = 0;
		          arg2 = 0;
		          obj = null;
		          replyTo = null;
		          sendingUid = UID_NONE;
		          workSourceUid = UID_NONE;
		          when = 0;
		          target = null;
		          callback = null;
		          data = null;
		  
		          synchronized (sPoolSync) {
		              if (sPoolSize < MAX_POOL_SIZE) {
		                  next = sPool;
		                  sPool = this;
		                  sPoolSize++;
		              }
		          }
		      }
		  ```
- 3、再获取Message时，不需要new  而是Obtain（）,从池子中取
  collapsed:: true
	- ```JAVA
	      public static Message obtain() {
	          synchronized (sPoolSync) {
	              if (sPool != null) {
	                  Message m = sPool;
	                  sPool = m.next;
	                  m.next = null;
	                  m.flags = 0; // clear in-use flag
	                  sPoolSize--;
	                  return m;
	              }
	          }
	          return new Message();
	      }
	  ```