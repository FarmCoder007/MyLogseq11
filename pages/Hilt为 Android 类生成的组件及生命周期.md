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
- # 组件
	- |  == Component ==   | ==注入对象==  | ==作用域==  | ==创建于==  |
	  |  SingletonComponent  | Application  |@Singleton|Application#onCreate()|
	  |  ActivityRetainedComponent  | 不适用  |@ActivityRetainedScoped|Activity#onCreate()|
	  |  ViewModelComponent  | ViewModel  |@ViewModelScoped|ViewModel created| 
	  |  ActivityComponent  | Activity  |@ActivityScoped|Activity#onCreate()|
	  |  FragmentComponent  | Fragment  |@FragmentScoped|Fragment#onAttach()|
	  |  ViewComponent  | View  |@ViewScoped|View#super()|
	  |  ViewWithFragmentComponent  | View with @WithFragmentBindings  |@ViewScoped|View#super()|
	  |  ServiceComponent  | Service  |@ServiceScoped|Service#onCreate()|
-