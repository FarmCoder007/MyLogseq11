- 1、Looper会维持一个MessageQueue，一个线程只有一个Looper【ThreadLocal保证的】
- 2、那么一个线程也只有一个MessageQueue
- ```java
      final MessageQueue mQueue;
      private Looper(boolean quitAllowed) {
          mQueue = new MessageQueue(quitAllowed);
          mThread = Thread.currentThread();
      }
  ```、
- 创建Looper的时候就创建了MessageQueue