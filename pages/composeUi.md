- ## 使用Compose写UI
	- compose是android新推出的UI工具包，使用可组合函数以声明式来构建UI，不再使用xml布局文件
	- 使用compose构建ui的例子：
	  collapsed:: true
		- ![image.png](../assets/image_1684391017018_0.png)
		-
	- 运行效果图
	  collapsed:: true
		- ![image.png](../assets/image_1684391037897_0.png)
	- 既然Compose是全新的一套UI工具包，那么它是如何构建UI的，是否用到了诸如LinearLayout、RelativeLayout、Button、Text这些android view呢？
	- 让我们来分析上图的布局
	- 遍历view，输入如下：
	  collapsed:: true
		- ![image.png](../assets/image_1684391051742_0.png)
	-