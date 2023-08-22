## 源码
	- ```java
	      static class ObserverWithState {
	          State mState;
	          LifecycleEventObserver mLifecycleObserver;
	  
	          ObserverWithState(LifecycleObserver observer, State initialState) {
	              mLifecycleObserver = Lifecycling.lifecycleEventObserver(observer);
	              mState = initialState;
	          }
	  
	          void dispatchEvent(LifecycleOwner owner, Event event) {
	              State newState = event.getTargetState();
	              mState = min(mState, newState);
	              mLifecycleObserver.onStateChanged(owner, event);
	              mState = newState;
	          }
	      }
	  ```
- 1、可以看到mLifecycleObserver会在ObserverWithState构造方法实例化。通过调用Lifecycling.lifecycleEventObserver方法获取LifecycleEventObserver实例；
  而ObserverWithState的实例化是在调用addObserver方法添加观察者的时候自动创建的
	- 就是 我们在Activity通过getLifecycle().addObserver(xxx)添加的自定义观察者的addObserver方法
- 2、主要是这里，调用观察者LifecycleEventObserver的onStateChanged方法
	- 首先看怎么根据LifecycleObserver 获取的[[LifecycleEventObserver]]
		-