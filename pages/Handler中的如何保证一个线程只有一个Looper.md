- 1、每个Thread都有个[[ThreadLocalMap]]的成员变量，不同线程 map不同
- 2、ThreadLocalMap是存储Entry（key value的）key 为ThreadLocal的弱引用
- 3、Looper中持有static final 修饰的ThreadLocal 对象引用，[[#red]]==保证全局唯一==、
	- ```java
	   static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
	  ```
- 4、那么在Looper.prepare（）时,先通过当前线程，根据唯一key去ThreadLocalMap获取是否有looper，有就抛出异常，没有就创建一个存进去
	- Looper.prepare()
	  collapsed:: true
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
	  collapsed:: true
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
	  collapsed:: true
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