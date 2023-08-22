## 1、Lifecycling.lifecycleEventObserver 返回的FullLifecycleObserverAdapter
	- androidx.lifecycle.Lifecycling.java
		- ```java
		     static LifecycleEventObserver lifecycleEventObserver(Object object) {
		          boolean isLifecycleEventObserver = object instanceof LifecycleEventObserver;
		          boolean isFullLifecycleObserver = object instanceof FullLifecycleObserver;
		          if (isLifecycleEventObserver && isFullLifecycleObserver) {
		              return new FullLifecycleObserverAdapter((FullLifecycleObserver) object,
		                      (LifecycleEventObserver) object);
		          }
		          if (isFullLifecycleObserver) {
		              return new FullLifecycleObserverAdapter((FullLifecycleObserver) object, null);
		          }
		  
		          if (isLifecycleEventObserver) {
		              return (LifecycleEventObserver) object;
		          }
		  
		          final Class<?> klass = object.getClass();
		          int type = getObserverConstructorType(klass);
		          if (type == GENERATED_CALLBACK) {
		              List<Constructor<? extends GeneratedAdapter>> constructors =
		                      sClassToAdapters.get(klass);
		              if (constructors.size() == 1) {
		                  GeneratedAdapter generatedAdapter = createGeneratedAdapter(
		                          constructors.get(0), object);
		                  return new SingleGeneratedAdapterObserver(generatedAdapter);
		              }
		              GeneratedAdapter[] adapters = new GeneratedAdapter[constructors.size()];
		              for (int i = 0; i < constructors.size(); i++) {
		                  adapters[i] = createGeneratedAdapter(constructors.get(i), object);
		              }
		              return new CompositeGeneratedAdaptersObserver(adapters);
		          }
		          return new ReflectiveGenericLifecycleObserver(object);
		      }
		  ```
	- 此方法通过判断观察者类型来转换对应的观察者适配器；
	- object 为自定义的观察者。官方推荐实现DefaultLifecycleObserver自定义观察者类， DefaultLifecycleObserver的父类就是FullLifecycleObserver。
	- 上边判断如果是FullLifecycleObserver，创建FullLifecycleObserverAdapter
- ## 2、现在我们主要看下FullLifecycleObserverAdapter适配器怎么实现的onStateChanged。
	- androidx.lifecycle.FullLifecycleObserverAdapter.java
	  collapsed:: true
		- ```java
		  class FullLifecycleObserverAdapter implements LifecycleEventObserver {
		  
		      private final FullLifecycleObserver mFullLifecycleObserver;
		      private final LifecycleEventObserver mLifecycleEventObserver;
		  
		      FullLifecycleObserverAdapter(FullLifecycleObserver fullLifecycleObserver,
		              LifecycleEventObserver lifecycleEventObserver) {
		          mFullLifecycleObserver = fullLifecycleObserver;
		          mLifecycleEventObserver = lifecycleEventObserver;
		      }
		  
		      @Override
		      public void onStateChanged(@NonNull LifecycleOwner source, @NonNull Lifecycle.Event event) {
		          switch (event) {
		              case ON_CREATE:
		                  mFullLifecycleObserver.onCreate(source);
		                  break;
		              case ON_START:
		                  mFullLifecycleObserver.onStart(source);
		                  break;
		              case ON_RESUME:
		                  mFullLifecycleObserver.onResume(source);
		                  break;
		              case ON_PAUSE:
		                  mFullLifecycleObserver.onPause(source);
		                  break;
		              case ON_STOP:
		                  mFullLifecycleObserver.onStop(source);
		                  break;
		              case ON_DESTROY:
		                  mFullLifecycleObserver.onDestroy(source);
		                  break;
		              case ON_ANY:
		                  throw new IllegalArgumentException("ON_ANY must not been send by anybody");
		          }
		          if (mLifecycleEventObserver != null) {
		              mLifecycleEventObserver.onStateChanged(source, event);
		          }
		      }
		  }
		  ```
	- 在onStateChanged中判断事件类型，直接调用了观察者的生命周期回调方法。