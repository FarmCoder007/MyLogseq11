- 子线程中持有主线程的handler -> sendMessage   -> messasgeQueue.enqueueMessage   //消息队列队列的插入节点 
                                   
  looper.loop()->  messasgeQueue.next（）->  handler.dispatchMessage()   ->   主线程的处理函数handler.handleMessage（）
-
- 总结：
	- 整个是Message（new 、Obtain）对象在动的过程，相当于Message这个内存在动，内存共享了