#  LifecycleRegistry 是 Lifecycle的子类
- ## 在Activity/Fragment中 自动创建了，通过getLifecycle 去获取对象，调用注册观察者
- ## 在Activity/Fragment中 getLifecycle()返回值
	- ```java
	      @NonNull
	      @Override
	      public Lifecycle getLifecycle() {
	          return mLifecycleRegistry;
	      }
	  ```
- # 处理生命周期事件
	- # [[LifecycleRegistry-handleLifecycleEvent]]