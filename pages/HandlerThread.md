# 一、概念
	- **HandlerThread****是****Thread****的子类，严格意义上来说就是一个线程，只是它在自己的线程里面帮我们创建了****Looper**
- # 二、HandlerThread存在的意义#card
	- ## 1、方便使用：a. 方便初始化，b，方便获取线程looper
	- ## 2、==保证了线程安全==
	  collapsed:: true
		- 我们一般在Thread里面 线程Looper进行初始化的代码里面，必须要对Looper.prepare(),同时要调用Loop.loop（）；
		- run方法
			- ```java
			  @Override
			  public void run() {
			  Looper.prepare();
			  Looper.loop();
			  }
			  ```
		- 而我们要使用子线程中的Looper的方式是怎样的呢？看下面的代码
			- ```java
			  Thread thread = new Thread(new Runnable() {
			  	Looper looper;
			  	@Override
			  	public void run() {
			  		// Log.d(TAG, "click2: " + Thread.currentThread().getName());
			  		Looper.prepare();
			  		looper =Looper.myLooper();
			  		Looper.loop();
			  	}
			  	public Looper getLooper() {
			  		return looper;
			  	}
			  });
			  thread.start();
			  Handler handler = new Handler(thread.getLooper());
			  ```
		- 上面这段代码有没有问题呢？
		  collapsed:: true
			- 1)在初始化子线程的handler的时候，我们无法将子线程的looper传值给Handler,解决办法有如下办法：
				- a. 可以将Handler的初始化放到 Thread里面进行
				- b. 可以创建一个独立的类继承Thread，然后，通过类的对象获取。
				- 这两种办法都可以，但是，这个工作 ==HandlerThread帮我们完成了==
			- 2）依据多线程的工作原理，我们在上面的代码中，调用 thread.getLooper（）的时候，此时的looper可能还没有初始化，此时是不是可能会挂掉呢？
		- HandlerThread 已经帮我们完美的解决了，这就是 handlerThread存在的必要性了。
- # 三、源码
  collapsed:: true
	- ```java
	  public void run() {
	  	mTid = Process.myTid();
	  	Looper.prepare();
	  	synchronized (this) {
	  		mLooper = Looper.myLooper();
	  		notifyAll(); //此时唤醒其他等待的锁，但是
	  	}
	  	Process.setThreadPriority(mPriority);
	  	onLooperPrepared();
	  	Looper.loop();
	  	mTid = -1;
	  }
	  ```
	- 它的==优点==就在于它的多线程操作，可以帮我们保证==使用Thread的handler时一定是安全的==。
	- 获取Looper的时候，如果未初始化完成会等待，释放锁
		- ```java
		      public Looper getLooper() {
		          if (!isAlive()) {
		              return null;
		          }
		          
		          // If the thread has been started, wait until the looper has been created.
		          synchronized (this) {
		              while (isAlive() && mLooper == null) {
		                  try {
		                      wait();
		                  } catch (InterruptedException e) {
		                  }
		              }
		          }
		          return mLooper;
		      }
		  
		  ```