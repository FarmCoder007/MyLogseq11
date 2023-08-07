- [文字绘制补充.xmind](../assets/文字绘制补充_1691242914125_0.xmind)
- ## [自定义view-文字测量](https://blog.csdn.net/xuwb123xuwb/article/details/114991330)
- ## 1、并不是所有的字体单位都用sp
	- dp 只与屏幕密度有关系   sp与用户设置（手机系统可以改字体大小） 和 屏幕密度有关
	- 当我们想设置的文字 不能让用户设置大小的时候 用dp        需要用户改变用sp     dp  sp  都可以根据屏幕密度适配
- ## 2、isFakeBoldText = true 是否加粗
  collapsed:: true
	- （fake 假加粗 android 通过自己算法 描粗的）
	- ## 加粗前
	  collapsed:: true
		- ![](https://img-blog.csdnimg.cn/20210318212830637.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1d2IxMjN4dXdi,size_16,color_FFFFFF,t_70)
	- ## 加粗后
	  collapsed:: true
		- ![](https://img-blog.csdnimg.cn/20210318212859974.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1d2IxMjN4dXdi,size_16,color_FFFFFF,t_70)
- ## 3、textAlign：文本对齐方式      只是 横向来说
  collapsed:: true
	- 1 默认是 textAlign = Paint.Align.LEFT，绘制起始是圆心，默认靠绘制点左侧对其
	  collapsed:: true
		- ![](https://img-blog.csdnimg.cn/20210318213113360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1d2IxMjN4dXdi,size_16,color_FFFFFF,t_70)
	- 2、textAlign = Paint.Align.RIGHT     靠绘制点的右侧对齐
	  collapsed:: true
		- ![](https://img-blog.csdnimg.cn/20210318213242502.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1d2IxMjN4dXdi,size_16,color_FFFFFF,t_70)
	- 3、textAlign = Paint.Align.CENTER    左右居中绘制
	  collapsed:: true
		- ![](https://img-blog.csdnimg.cn/20210318213336450.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1d2IxMjN4dXdi,size_16,color_FFFFFF,t_70)
- ## 4、绘制bitmap或者图片的 起始点  与  绘制文字起始点的不同
	- 4-1、绘制bitmap时设置的 起始点 是 bitmap的左上方
		- ![](https://img-blog.csdnimg.cn/20210318213623382.png)
	- 4-2、绘制文字时  设置的 起始点
		- 这个起始点 分 横向 纵向来分
		- ## 横向上起始点 ：
			- 如 三里  三个配置  分别位于文字的  左侧  右侧  中间
		- ## [[#red]]==**纵向上设置的起始点位于文字的基线上 baseline**==
			- ![](https://img-blog.csdnimg.cn/20210318213940601.png)
		- 所以 我们正常绘制的时候  文字 会 在我们想要的纵向偏上      因为 中心点在基线上    那么我们怎么修正
- ## 5、[[静态文字-纵向修正文字]]：找出偏移   -  偏移
- ## 6、[[动态文字-纵向校正]]
- ## 7、文字贴边，左贴边，上贴边
	- ## 情景一   把文字 绘制到左上角   不修正位置的情况下 左对齐
	  collapsed:: true
		- ```java
		  // 绘制文字2
		          paint.textAlign = Paint.Align.LEFT
		          canvas.drawText("abab",0f,0f,paint)
		  ```
		- ![](https://img-blog.csdnimg.cn/2021031822213121.png)
		- 其实是绘制上的 ，显示的位置 是基线的位置，上边学到 不修正  0,0处是基线的位置   文字 在 基线之上
		- ![](https://img-blog.csdnimg.cn/20210318222521360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1d2IxMjN4dXdi,size_16,color_FFFFFF,t_70)
		- ## 解决1:推荐,把文字顶部贴顶：[[#red]]==**把  baseline 与 top的距离 做向下偏移  【阅读型可以用】**==
		  collapsed:: true
			- ```java
			  // 绘制文字2
			          paint.textAlign = Paint.Align.LEFT
			          // 测量ascent  descent后放入fontMatrix
			          paint.getFontMetrics(fontMatrix)
			          // 纵向修正 baseline 到 top  之间的距离  其实就是 0-fontMatrix.top
			          canvas.drawText("abab",0f,-fontMatrix.top,paint)
			  ```
			- ![](https://img-blog.csdnimg.cn/20210318222920918.png)
			- 上边的空隙就是top
		- ## 解决2  使用      getTextBounds   做修正 会紧贴上   看起来不太舒服[[#red]]==**【观赏型可以贴边 】**==
		  collapsed:: true
			- ```java
			   
			          paint.textAlign = Paint.Align.LEFT
			          // 测量ascent  descent后放入fontMatrix
			          paint.getTextBounds("abab",0,"abab".length,bounds)
			          canvas.drawText("abab",0f, (-bounds.top).toFloat(),paint)
			  ```
			- ![](https://img-blog.csdnimg.cn/20210318223045625.png)
- ## 8、左贴边
  collapsed:: true
	- 虽然   设置paint.textAlign = Paint.Align.LEFT   可以实现左贴边     但是 多行 上下  文字大小不一样的时候  会有问题
		- ```java
		   // 绘制文字2
		          paint.textAlign = Paint.Align.LEFT
		          paint.textSize=150.dp
		          // 测量ascent  descent后放入fontMatrix
		          paint.getFontMetrics(fontMatrix)
		          // 纵向修正 baseline 到 top  之间的距离  其实就是 0-fontMatrix.top
		          paint.getTextBounds("abab",0,"abab".length,bounds)
		          canvas.drawText("abab",0f, (-bounds.top).toFloat(),paint)
		   
		          // 绘制文字3
		          paint.textSize=15.dp
		          // 纵向修正 baseline 到 top  之间的距离  其实就是 0-fontMatrix.top
		          paint.getTextBounds("abab",0,"abab".length,bounds)
		          canvas.drawText("abab",0f, (-bounds.top).toFloat(),paint)
		  ```
	- ![](https://img-blog.csdnimg.cn/20210318223625839.png)
	- 都是贴左  贴顶   但是   由上可见 他们字体大的  左侧 和 ab之间会有间隙的
	- ![](https://img-blog.csdnimg.cn/20210318224022603.png)
	- 此时因为 textBounds 是在 起始点的右侧的   那么   可以用 textBounds.left修正
	- 修正后
	- ```java
	   // 绘制文字2
	          paint.textAlign = Paint.Align.LEFT
	          paint.textSize=150.dp
	          // 测量ascent  descent后放入fontMatrix
	          paint.getFontMetrics(fontMatrix)
	          // 纵向修正 baseline 到 top  之间的距离  其实就是 0-fontMatrix.top
	          paint.getTextBounds("abab",0,"abab".length,bounds)
	          canvas.drawText("abab", (-bounds.left).toFloat(), (-bounds.top).toFloat(),paint)
	   
	          // 绘制文字3
	          paint.textSize=15.dp
	          // 纵向修正 baseline 到 top  之间的距离  其实就是 0-fontMatrix.top
	          paint.getTextBounds("abab",0,"abab".length,bounds)
	          canvas.drawText("abab",(-bounds.left).toFloat(), (-bounds.top).toFloat(),paint)
	  ```
	- ![](https://img-blog.csdnimg.cn/20210318224201868.png)
- ## 9、[[多行绘制]]
-