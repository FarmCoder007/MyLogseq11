title:: View.java的dispatchTouchEvent

- 1、最先调用的地方，view设置了[[#red]]==**mOnTouchListener**==，先走它的onTouch。看其返回值
	- 返回true ：被onTouch消费  不会继续分发Down事件
	- 返回false：这时才会执行到TextVIew的onTouchEvent()方法
- 2、（ 1、view没有写setOnTouchListener2、要么写了返回false 没有消费）下边才会走[[#red]]==**onTouchEvent**==(event)
- 3、Action_Down事件里,会判断==**onLongClickListener**==
- 4、Action_UP判断如果设置了[[#red]]==**mOnClickListener**==，则会走onClick方法。同时设置result = true 告诉父容器消费了