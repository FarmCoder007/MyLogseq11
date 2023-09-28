- 源码
	- ```java
	  private boolean updateRegistration(int localFieldId, Object observable,
	              CreateWeakListener listenerCreator) {
	          if (observable == null) {
	              return unregisterFrom(localFieldId);
	          }
	          WeakListener listener = mLocalFieldObservers[localFieldId];
	          if (listener == null) {
	              registerTo(localFieldId, observable, listenerCreator);
	              return true;
	          }
	          if (listener.getTarget() == observable) {
	              return false;//nothing to do, same object
	          }
	          unregisterFrom(localFieldId);
	          registerTo(localFieldId, observable, listenerCreator);
	          return true; 
	      }
	  ```
- 参数
	- localFieldId: 为BR 文件里的id.类里每生成一个属性，就会带一个这个id
		- ![image.png](../assets/image_1691657414272_0.png)
	- observable:为被观察者。上边传入的User
	- 传入了一个 CREATE_PROPERTY_LISTENER 创建属性的监听器
		- ```java
		  private static final CreateWeakListener CREATE_PROPERTY_LISTENER = new CreateWeakListener() {
		    @Override
		    public WeakListener create(ViewDataBinding viewDataBinding, int localFieldId) {
		      // 返回一个弱引用的属性监听
		      return new WeakPropertyListener(viewDataBinding, localFieldId).getListener();
		    }
		  };
		  ```
- ## 总结
	- 1、User类被观察者继承BaseObserver
	- 2、BaseObserver持有 mCallbacks 集合。存的是WeakPropertyListener（实现了OnPropertyChangedCallback）
	- 3、WeakPropertyListener 持有 WeakListener，WeakListener持有ViewDataBinding。
	- 4、ViewDataBinding 持有 每个属性的监听器WeakListener
	- > 小结。我们的viewModel 或者说User持有了ViewDataBinding 。间接持有每个属性监听器