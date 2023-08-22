# 写ANR
	- ## [[#red]]==**原因：**==
		- Android系统在页面切换前，将数据写入文件。在ActivityThread的handlePauseActivity和handleStopActivity时会通过waitToFinish保证这些异步任务都已经被执行完成。如果这个时候[[#red]]==**过渡使用apply方法**==，则可能导致onpause，onStop执行时间较长，从而导致ANR。
	- handlePauseActivity
	  collapsed:: true
		- ```java
		  private void handlePauseActivity(IBinder token, boolean finished,
		            boolean userLeaving, int configChanges, boolean dontReport, int seq) {
		       ......
		            r.activity.mConfigChangeFlags |= configChanges;
		            performPauseActivity(token, finished, r.isPreHoneycomb(), "handlePauseActivity");
		  
		            // Make sure any pending writes are now committed.
		            if (r.isPreHoneycomb()) {
		                QueuedWork.waitToFinish();
		            }
		           ......
		    }
		  ```
	- **waitToFinish**
	  collapsed:: true
		- ```java
		  public static void waitToFinish() {
		        Handler handler = getHandler();
		        try {
		            processPendingWork();
		        } finally {
		            StrictMode.setThreadPolicy(oldPolicy);
		        }
		  
		        try {
		            while (true) {
		                Runnable finisher;
		  
		                synchronized (sLock) {
		                    finisher = sFinishers.poll();
		                }
		  
		                if (finisher == null) {
		                    break;
		                }
		  
		                finisher.run();
		            }
		        } finally {
		            sCanDelay = true;
		        }
		    }
		  ```
	- 主线程执行processPendingWork（）方法，把之前未执行完的内容存储到文件的操作执行完，这部分动作直接在主线程执行，如果有未执行的文件操作并且文件较大，则主线程会因为IO时间长造成ANR。
	- 循环取出sFinishers数组，执行他的run方法。如果这时候有多个异步线程或者异步线程时间过长，同样会造成阻塞产生ANR。。
	- >就是调用CountDownLatch对象的await方法，阻塞当前线程，将当前线程加入阻塞队列。所以如果过渡使用apply方法，则可能导致onpause，onStop执行时间较长从而导致ANR。
- # 首次读操作的ANR
	- ## [[#red]]==**原因：**==
		- sp 文件创建以后，会单独的使用一个线程来加载解析对应的 sp 文件。但是当 UI 线程尝试访问 sp 中内容时，如果 sp 文件还未被完全加载解析到内存，此时 UI 线程会被 block，直到 SP 文件被完全加载到内存中为止
	- sp 被创建的时候会同时启动一个线程加载对应的 sp 文件，执行 startLoadFromDisk();
	  在 startLoadFromDisk()时，标记 sp 不可使用状态，后期无论是尝试读数据或者写数据，读写线程都会被 block，直到 sp 文件被全部加载解析完毕。
	- 作者：android_greenhand
	  链接：https://juejin.cn/post/7221020355292315709
	  来源：稀土掘金
	  著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。