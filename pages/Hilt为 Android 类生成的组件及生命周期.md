# 背景
	- 一般情况下，我们在设置注入对象时仅使用**@Inject**来标记，此时注入对象是无作用域的，这意味着==**每次都会注入一个新的对象**==。
		- 如果我们在注入方法的上面添加Scope相关的注解，作用域绑定只会在他作用域范围内的组件中生成一个实例，举个例子来说
		- ```kotlin
		  // 仅有Inject注解注入是无作用域的，每一次请求都会创建一个新的实例
		  class UnscopedBinding @Inject constructor() {
		  }
		  
		  //有ActivityScoped注解修饰的对象，在同一个Activity中的实例是唯一的。
		  @ActivityScoped
		  class ScopedBinding @Inject constructor() {
		  }
		  ```