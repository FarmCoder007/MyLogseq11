# 背景
collapsed:: true
	- 一般情况下，我们在设置注入对象时仅使用**@Inject**来标记，此时注入对象是无作用域的，这意味着==**每次都会注入一个新的对象**==。
	- 如果我们在注入方法的上面添加Scope相关的注解，[[#red]]==**作用域绑定只会在他作用域范围内的组件中生成一个实例，举个例子来说**==
		- ```kotlin
		  // 仅有Inject注解注入是无作用域的，每一次请求都会创建一个新的实例
		  class UnscopedBinding @Inject constructor() {
		  }
		  
		  //有ActivityScoped注解修饰的对象，在同一个Activity中的实例是唯一的,不同Activity才会创建
		  @ActivityScoped
		  class ScopedBinding @Inject constructor() {
		  }
		  ```
	- 有ActivityScoped注解修饰的对象，在同一个Activity中的实例是唯一的，不同的Activity还是会创建新的对象。
	- 从上面的表格中可以看到Hilt对于注入对象的作用域支持很多：有常见的全局唯一单例类型的对象，也有和ViewModel声明周期对应的作用域。不同的作用域这样就可解决多对象单例过多占用资源的问题。
- # 组件及对应的生命周期
	- Hilt 会按照相应 Android 类的生命周期自动创建和销毁生成的组件类的实例。
	- |  == Component ==   | ==注入对象==  | ==作用域==  | ==创建于==  |==销毁时机==|
	  |  SingletonComponent  | Application  |@Singleton|Application#onCreate()|`Application` 已销毁|
	  |  ActivityRetainedComponent  | 不适用  |@ActivityRetainedScoped|Activity#onCreate()|Activity#onDestroy()|
	  |  ViewModelComponent  | ViewModel  |@ViewModelScoped|ViewModel created| `ViewModel` 已销毁|
	  |  ActivityComponent  | Activity  |@ActivityScoped|Activity#onCreate()|Activity#onDestroy()|
	  |  FragmentComponent  | Fragment  |@FragmentScoped|Fragment#onAttach()|Fragment#onDestroy()|
	  |  ViewComponent  | View  |@ViewScoped|View#super()|`View` 已销毁|
	  |  ViewWithFragmentComponent  | View with @WithFragmentBindings  |@ViewScoped|View#super()|`View` 已销毁|
	  |  ServiceComponent  | Service  |@ServiceScoped|Service#onCreate()|Service#onDestroy()|
- # 注
	- 1、Hilt 不会为广播接收器生成组件，因为 Hilt 直接从 `SingletonComponent` 注入广播接收器
	- 2、`ActivityRetainedComponent` 在配置更改后仍然存在，因此它在第一次调用 `Activity#onCreate()` 时创建，在最后一次调用 `Activity#onDestroy()` 时销毁。