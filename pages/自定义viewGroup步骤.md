# 自定义viewGroup步骤#card
	- ## 一、重写onMeasure（int widthMeasureSpec, int heightMeasureSpec）
		- 1、遍历每个子view ,getChildMeasureSpec() 结合viewGroup的测量模式 和 子view的layoutParams测量子view
		- 2、测量完成后，得出子view的实际位置和尺寸，并暂时保存
		- 3、根据子view的尺寸，并且计算出自己的尺寸，并用setMeasuredDimension(width，height)保存
	- ## 二、重写onLayout()
		- 遍历每个子view  调用它们的layout()方法来将位置和尺寸传给它们