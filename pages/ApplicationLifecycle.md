- 不需要分发空白Fragment 生命周期 回调。注册监听直接互调onStart
- ## 源码
	- ```java
	  class ApplicationLifecycle implements Lifecycle {
	    @Override
	    public void addListener(@NonNull LifecycleListener listener) {
	      listener.onStart();
	    }
	  
	    @Override
	    public void removeListener(@NonNull LifecycleListener listener) {
	      // Do nothing.
	    }
	  }
	  ```