- 首先：==内存是没有线程之分的==，比如Activity中new一个集合，在子线程可以用在主线程也可以用
	- 子线程：执行run里的函数，就是在子线程运行的
- ## 示例代码
  collapsed:: true
	- ```java
	    // 主线程的handler
	    Handler mHandler = new Handler(){
	          @Override
	          public void handleMessage(@NonNull Message msg) {
	              super.handleMessage(msg);
	          }
	      };
	  
	      // 子线程发送消息
	      public void click(View view){
	          new Thread(new Runnable() {
	              @Override
	              public void run() {
	                  Looper.prepare();
	  
	                  Looper.loop();
	                  // 子线程中 调用主线程的handler 发送消息
	                  mHandler.sendMessage(Message.obtain());
	              }
	          }).start();
	          // 主线程中 调用 子线程的handler 发送消息 怎么到子线程接受的
	          threadHandler.sendMessage(Message.obtain());
	      }
	  
	      Handler threadHandler;
	  
	  
	      // 子线程
	      public void test(){
	          new Thread(new Runnable() {
	              @Override
	              public void run() {
	                  if(threadHandler == null){
	                      Looper.prepare();
	                      // 子线程中的handler 处理
	                      threadHandler = new Handler(){
	                          @Override
	                          public void handleMessage(@NonNull Message msg) {
	                              super.handleMessage(msg);
	                          }
	                      };
	                      Looper.loop();
	                  }
	              }
	          }).start();
	      }
	  ```
- ## 顺序
	- 1、由上述代码click函数中，子线程使用主线程的handler发送消息[[#red]]==子线程调度==
	  collapsed:: true
		- ```java
		   // 子线程中 调用主线程的handler 发送消息
		   mHandler.sendMessage(Message.obtain());
		  ```
	- 2、由[[机制构成]]Handler的函数可知，最终调用到MessageQueue.enqueueMessage()函数，加入到主线程创建的消息队列中[[#red]]==子线程调度==
	- 3、主线程的Looper不断轮训从主线程创建的消息队列取消息。见[[Looper]].loop()函数。==主线程运行==
	- 4、MessageQueue.next()取出消息也是在==主线程运行==
	- 5、由Loop()函数可知取出的消息，会调度msg.target.dispatchMessage(msg);进行分发（target是调用sendMsg的那个handler）==主线程运行==
	  collapsed:: true
		- ```java
		      public void dispatchMessage(@NonNull Message msg) {
		          if (msg.callback != null) {
		              handleCallback(msg);
		          } else {
		              if (mCallback != null) {
		                  if (mCallback.handleMessage(msg)) {
		                      return;
		                  }
		              }
		              handleMessage(msg);
		          }
		      }
		  ```
	- 6、调用到Handler 的handleMessage方法。完成了消息的传递==主线程运行==