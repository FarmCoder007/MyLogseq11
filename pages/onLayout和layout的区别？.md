# onLayout和layout的区别？#card
	- onLayout
		- 只有viewGroup才需要 重写 onLayout方法 在这里 布局子view   把 每个子view的上下左右通layout传给它，，让子view保存起来
	- layout(left,top,right,bittom)   是你父view 传给你的位置，让你去保存的
-