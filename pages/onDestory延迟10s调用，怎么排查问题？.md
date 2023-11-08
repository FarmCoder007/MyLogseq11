# onDestory延迟10s调用，怎么排查问题？#Card
	- 虽然有兜底机制，但无论如何这肯定不是我们想看到的。如果我们项目中的 onStop/onDestroy 延迟了 10s 调用，该如何排查问题呢？可以利用 [[#red]]==**Looper.getMainLooper().setMessageLogging()**== 方法，打印出主线程消息队列中的消息。每处理一条消息，都会打印如下内容：
	- ```java
	  logging.println(">>>>> Dispatching to " + msg.target + " " + msg.callback + ": " + msg.what);
	  logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
	  ```
	- 另外，由于 `onStop/onDestroy` 调用时机的不确定性，在做资源释放等操作的时候，一定要考虑好，以避免产生资源没有及时释放的情况。