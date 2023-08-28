# 基础知识
collapsed:: true
	- ## 1、[[Android坐标系总结-面试]]
	- ## 2、[[getMeasuredWidth() 和 getWidth() 区别 和使用场景]]
	- ## 3、[[自定义View的基本方法]]
	- ## 4、自定义view和ViewGroup需要重新的方法
	  collapsed:: true
		- 自定义View: 只需要重写onMeasure()和onDraw()
		- 自定义ViewGroup: 则只需要重写onMeasure()和onLayout()，也可以重新onDraw绘制分割线
		- [[自定义view需要重新的方法-面试]]
	- ## 5、[[View构造函数+使用场景]]
	- ## 6、布局文件xml的键值对，在inflate函数解析的
	- ## 8、[[自定义view的绘制流程-生命周期方法]]
	- ## 4、[[onLayout和layout的区别？]]
	- ## 8、自定义view中为什么要Measure？
	  collapsed:: true
		- 为了给子view分配空间，子view要显示，需要结合自己layoutParams设置的大小，和父view规定的测量模式。去计算测量
	- ## 9、invalidate和 requestLayout区别？
		- invalidate布局没变化，刷新数据Ondraw
		- requestLayout,布局有变化，重新布局onMeasure onLayout onDraw
	- ## 10、刷新中Invalidate和postInvalidate的区别
		- invalidate只能在ui线程使用
		- postInvalidate可以在子线程中使用，内部也是用了handler
	- ## 11、 invalidate()和requestLayout()触发的生命周期
		- ![](https://img-blog.csdn.net/20160826104112634?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
- # 绘制文字
	- ## 绘制文字，纵向起始位置
		- 纵向上设置的起始点位于文字的基线上 baseline、
		- 所以你drawText（“xxx”,0,0,paint）想从00 开始绘制时，实际baseline在 00 那条线，实际文字就在屏幕上方外部了
	- ## 9、[[自定义view-文字基线]]
	- ## 5、[[静态文字-纵向修正文字]]：找出偏移   -  偏移
	- ## 6、[[动态文字-纵向校正]]
- # 原理流程
	- ## [[自定义viewGroup-面试题]]
	- ## 7、[[MeasureSpec-面试题]]
	- ## 3、[[view的布局流程（测量 布局）]]，绘制就是调用canvas api了
	- ## [[View的绘制流程]]触发从setCopntetnView
	- ## [[Activity布局流程/创建流程ActivityThread+PhoneWindow]]
- # 嵌套滑动
	- ## [[模仿淘宝首页滑动]]
- # 滑动冲突
	- ## [[滑动事件冲突]]
- # 怎么实现图片圆角
	- ## 1、使用Xfermode 转换模式SRC_IN
	  collapsed:: true
		- 1、canvas.saveLayer 先挖出一块区域做离屏缓冲
		- 2、先绘制圆心
		- 2、使用融合模式
			- paint.xfermode = PorterDuffXfermode(PorterDuff.Mode.SRC_IN)
		- 3、绘制Bitmap
		- 4、用完xfermode  置空
		- 5、 canvas.restoreToCount(count)把离屏缓冲中的合成后的图形放回绘制区域
		- ## 优化
			- 由于 Xfermode   绘制的时候需要看一个底图的      直接绘制 底图不只有第一个画的那个圆形，，还有白色的背景  和这个view底下的其他view    所以 需要单独只有那个圆的背景  就需要抽出一个层
			- Canvs.saveLayer() 把绘制区域拉到单独的离屏缓冲⾥，单独出来一个层  在用Xfermode     进行融合
			- 什么叫 离屏缓冲： 绘制的时候  在单独的一个地方（什么都没有  也就是透明的） 绘制  不在该view里绘制          绘制完了再放到该view的区域里
			- ⽤ Canvas.restoreToCount() 把离屏缓冲中的合成后的图形放回绘制区域
		- 代码
		  collapsed:: true
			- ```java
			  override fun onDraw(canvas: Canvas) {
			          /**
			           * 参数一 要挖出哪块区域做离屏缓冲    挖出 需要大小就行  离屏缓冲 很耗资源
			           * 离屏缓冲  拿出一块区域 进行绘制
			           */
			          val count  = canvas.saveLayer(bounds,null)
			   
			          // 先绘制个圆形
			          canvas.drawOval(
			              IMAGE_PADDING,
			              IMAGE_PADDING,
			              IMAGE_PADDING + IMAGE_WIDTH,
			              IMAGE_PADDING + IMAGE_WIDTH,
			              paint
			          )
			          // 改变paint.xfermode  改变融合模式  在绘制的时候就会结合上边的圆绘制
			          paint.xfermode = (XFERMODER)
			   
			          /**
			           * 给出左上角  确定 图片绘制的位置
			           */
			          canvas.drawBitmap(
			              getBitmapFromRes(IMAGE_WIDTH.toInt()),
			              IMAGE_PADDING,
			              IMAGE_PADDING,
			              paint
			          )
			          // 用完xfermode  置空
			          paint.xfermode = null
			                  /**
			           * 参数   把离屏缓冲中的合成后的图形放回绘制区域
			           */
			          canvas.restoreToCount(count)
			   
			      }
			  ```
	- ## 2、clipPath范围裁切裁出圆形也可以