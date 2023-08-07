## 1、ViewGroup的dispatchTouchEvent
collapsed:: true
	- 1、首先disallowIntercept，看子view有没有调用，说不允许拦截它的事件（默认为false的）
		- false:就是没有说不拦截。可以继续看下边ViewGroup的拦截事件onInterceptTouchEvent
		- true: 要求父view不拦截 和 onInterceptTouchEvent返回false一致
- ## 2、看viewGroup重写的onInterceptTouchEvent
  collapsed:: true
	- ### [[#red]]==**ture:.拦截**==（走自己的分发和处理，相当于自己是最后一个,看自己处不处理）
		- ```java
		   if (mFirstTouchTarget == null) {
		     // viewGroup 拦截后 处理的方法
		     handled = dispatchTransformedTouchEvent(ev, canceled, null,
		                                             TouchTarget.ALL_POINTER_IDS);
		   } 
		  ```
		- ### 进入[[dispatchTransformedTouchEvent]]首先viewGroup的拦截后 传入的child = null，
			- ```java
			  if (child == null) {
			    handled = super.dispatchTouchEvent(transformedEvent);
			  }
			  ```
			- 走的super.dispatchTouchEvent，父类的就是view.java。还是走view的分发那一套。如果viewGroup 重写了 onTouch,onTouchEvent，等方法就会进入，判断是否消费
			- 返回结果赋值给handled。判断是否消费了。相当于dispatchTransformedTouchEvent的返回值也是这个。最后拦截后dispatchTouchEvent的返回值也是这个
		- ### 拦截后的处理 接口 返回了handled，代表事件在viewGroup是否消费了
	- ### [[#red]]==**[[false:不拦截]]**==。
- ## 3、走子view的分发dispatchTouchEvent
  collapsed:: true
	- ### [[View的dispatchTouchEvent，主要是事件处理]]
-