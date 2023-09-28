# 对于每个view
- # 1、ViewGroup：onMeasure，测量自己和子view
	- ### 测量子view
		- 2-1、首先遍历子view，结合子view的LayoutParams 和 父view的测量模式，父view可用空间。通过[[#red]]==**getChildMeasureSpec 计算出子view的测量模式MeasureSpec**==
		- 2-2、[[#red]]==**调用子view的measure（）方法**== ，传入子view的MeasureSpec，让子view 自我测量
		- 2-3、[[#red]]==**子view在自己的onMeasure（）**==中，根据viewGroup传入的MeasureSpec 以及自己的实际想要的大小算出自己的期望尺寸，[[#red]]==**setMeasuredDimension进行保存**==
		  collapsed:: true
			- 【注意 子view 在onMeasure里不听父view的建议  父view 可以在onlayot的时候强行修正的】
			- 如果是ViewGroup，还会在这里调用每个子view的measure（）进行测量
	- ### 测量自身
		- 2-4、测量出所有子view的位置和尺寸后，并且计算出自己的尺寸，并用setMeasuredDimension(width，height)保存
- # 2、ViewGroup：OnLayout进行布局
	- 1、viewGroup 在onLayout方法中， 会调用子view的layout()方法，把子view的位置和尺寸传入 ；
	- 2、子view 在layou(）方法里将父view 传入的实际尺寸 和位置保存 【传的值就是左上右下】
		- 【子view 在layout(left,top,right,bittom)   里接收到父view 传给你的尺寸】
-
-
- ## 其他
	- 1、首先通过getLayoutParams,获取定义在xml里的宽高
	  collapsed:: true
		- 首先我们在xml定义View的宽高，测量的时候通过LayoutParams获取         【第一步是开发者的要求】
	-