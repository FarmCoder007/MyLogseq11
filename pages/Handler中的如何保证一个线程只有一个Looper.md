## 1、Looper是通过ThreadLocal存在当前线程的[[ThreadLocalMap]]私有变量中的
	- 每个线程都有自己的ThreadLocalMap。存储Entry（key value的）key 为ThreadLocal的弱引用
- ## 2、而Looper里有个static final修饰的 ThreadLocal ，==**全局唯一key**==作为 ThreadLocalMap存储的key
	- ```java
	   static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
	  ```
- ## 3、每个Looper都是通过prepare（）方法创建的（[[#red]]==**保证不重复创建**==）
	- 1、首先[[#red]]==**sThreadLocal.get()获取Looper对象**==。实际是先获取当前线程，拿到当前线程的ThreadLocalMap，根据唯一keyThreadLocal 取对应的Looper，有就抛出异常
	- 2、没有获取到调用 sThreadLocal.set(new Looper(quitAllowed));
		- new 一个 looper对象，通过ThreadLoacl 存入当前线程的ThreadLoaclmap里。key是全局唯一的static final 修饰的。
		- Looper唯一，MessageQueue在Looper构造函数里创建的也是唯一的
-
-
-
- ## 源码参考
  collapsed:: true
	- 4、那么在Looper.prepare（）时,先通过当前线程，根据唯一key去ThreadLocalMap获取是否有looper，有就抛出异常，没有就创建一个存进去
		- Looper.prepare()
			- ```java
			      private static void prepare(boolean quitAllowed) {
			          // 校验当前线程  是否存储过 looper
			          if (sThreadLocal.get() != null) {
			              throw new RuntimeException("Only one Looper may be created per thread");
			          }
			          sThreadLocal.set(new Looper(quitAllowed));
			      }
			  ```
		- ThreadLocal.get:获取当前线程的ThreadLocalMap（key为第3步）中存的looper
			- ```java
			      /**
			       * Returns the value in the current thread's copy of this
			       * thread-local variable.  If the variable has no value for the
			       * current thread, it is first initialized to the value returned
			       * by an invocation of the {@link #initialValue} method.
			       *
			       * @return the current thread's value of this thread-local
			       */
			      public T get() {
			          Thread t = Thread.currentThread();
			          ThreadLocalMap map = getMap(t);
			          if (map != null) {
			              ThreadLocalMap.Entry e = map.getEntry(this);
			              if (e != null) {
			                  @SuppressWarnings("unchecked")
			                  T result = (T)e.value;
			                  return result;
			              }
			          }
			          return setInitialValue();
			      }
			  ```
		- ThreadLocal.set：会把创建的looper，存在当前线程的ThreadLocalMap中，key是上边的唯一key
			- ```java
			      /**
			       * Sets the current thread's copy of this thread-local variable
			       * to the specified value.  Most subclasses will have no need to
			       * override this method, relying solely on the {@link #initialValue}
			       * method to set the values of thread-locals.
			       *
			       * @param value the value to be stored in the current thread's copy of
			       *        this thread-local.
			       */
			      public void set(T value) {
			          // 获取当前线程
			          Thread t = Thread.currentThread();
			          // 获取线程里的 ThreadLocalMap 存储
			          ThreadLocalMap map = getMap(t);
			          if (map != null)
			              map.set(this, value);
			          else
			              createMap(t, value);
			      }
			  
			  ```
	- 5、所以有就抛异常，没有存就创建根据3中的唯一key去存入，来保证value为当前线程的looper是唯一的，looper里对应MessageQueue也是唯一的
-
-
-
- # 总结
	- 在 Android 应用程序中sThreadLocal 是全局唯一的，用于当做唯一key[[#red]]==在每个线程的[[ThreadLocalMap]]存储每个线程的 Looper 实例==。
	- 而 Looper 是线程唯一的，每个线程只能有一个与之关联的 Looper 实例，用于维护该线程的消息队列和消息循环机制。