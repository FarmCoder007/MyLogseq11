## 1、介绍
	- 注重生命周期的方式存储和管理界面相关的数据,可在发生屏幕旋转等配置更改后继续留存
- ## 2、作用
	- 3、Fragment共享ViewModel。实现通信
	- 当系统配置发生变更时，比如横竖屏切换，切换字体字号 语言等。用于管理数据的保存与恢复,
	- 以前需要自己在saveInstanceState 里处理，现在ViewModel 可以自动处理
- ## 3、 [[ViewModel数据恢复原理总结]]
- ## 4、什么时候调用clear清空数据
	- 在ComponentActivity ，注册lifeCycle，观测Activity的生命周期。如果调用onDestroy时。就会调用clear
	- ```java
	          getLifecycle().addObserver(new LifecycleEventObserver() {
	              @Override
	              public void onStateChanged(@NonNull LifecycleOwner source,
	                      @NonNull Lifecycle.Event event) {
	                  if (event == Lifecycle.Event.ON_DESTROY) {
	                      // Clear out the available context
	                      mContextAwareHelper.clearAvailableContext();
	                      // And clear the ViewModelStore
	                      // 配置没变更才清除
	                      if (!isChangingConfigurations()) {
	                          getViewModelStore().clear();
	                      }
	                  }
	              }
	          });
	  ```