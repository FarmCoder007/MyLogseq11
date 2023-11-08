# MessageQueue，属于哪个线程？问队列创建流程#Card
	- ## 1、looper在哪个线程 MessageQueue就属于哪个线程。
	- ## 1、Looper会维持一个MessageQueue，一个线程只有一个Looper【ThreadLocal保证的】
	- ## 2、那么一个线程也只有一个MessageQueue
		- ```java
		      final MessageQueue mQueue;
		      private Looper(boolean quitAllowed) {
		          mQueue = new MessageQueue(quitAllowed);
		          mThread = Thread.currentThread();
		      }
		  ```、
	- ## 创建Looper的时候就创建了MessageQueue