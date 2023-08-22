## 代码
	- ```java
	  class ActivityFragmentLifecycle implements Lifecycle {
	    private final Set<LifecycleListener> lifecycleListeners =
	        Collections.newSetFromMap(new WeakHashMap<LifecycleListener, Boolean>());
	    private boolean isStarted;
	    private boolean isDestroyed;
	  
	    /**
	       首次默认先走onStop  再走onStart
	     */
	    @Override
	    public void addListener(@NonNull LifecycleListener listener) {
	      lifecycleListeners.add(listener);
	  
	      if (isDestroyed) {
	        listener.onDestroy();
	      } else if (isStarted) {
	        listener.onStart();
	      } else {
	        listener.onStop();
	      }
	    }
	  
	    @Override
	    public void removeListener(@NonNull LifecycleListener listener) {
	      lifecycleListeners.remove(listener);
	    }
	  
	    void onStart() {
	      isStarted = true;
	      for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {
	        lifecycleListener.onStart();
	      }
	    }
	  
	    void onStop() {
	      isStarted = false;
	      for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {
	        lifecycleListener.onStop();
	      }
	    }
	  
	    void onDestroy() {
	      isDestroyed = true;
	      for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {
	        lifecycleListener.onDestroy();
	      }
	    }
	  }
	  ```
- ## 作用
	- 1、set存储，注册的生命周期监听
	- 2、在空白Fragment，生命周期回调时，会调到这里。遍历监听器进行分发回调