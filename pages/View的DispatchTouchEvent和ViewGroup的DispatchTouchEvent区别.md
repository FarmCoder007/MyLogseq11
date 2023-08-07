## View的DispatchTouchEvent，[[#red]]==**主要内部处理事件**==
	- A: 我们知道 View 可以注册很多事件监听器，例如：单击事件(onClick)、长按事件(onLongClick)、触
	  摸事件(onTouch)，并且View自身也有 onTouchEvent 方法，那么问题来了，这么多与事件相关的方
	  法应该由谁管理？
	- 毋庸置疑就是 dispatchTouchEvent，所以 View 也会有事件分发。
- ## ViewGroup的DispatchTouchEvent
	- [[#red]]==**功能主要处理往子view事件分发**==
-