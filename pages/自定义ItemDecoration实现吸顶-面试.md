## 1、重写 getItemOffsets
	- ## 思路
		- 重写 getItemOffsets 设置itemView偏移，如果是分组的header 多预留空间,否则正常细分割线
- ## 2、重写OnDraw。将上方空出来的区域绘制分割线
	- ## 思路
		- 1、onDraw 是绘制在ItemView之后的。itemView可以遮挡，OnDraw的绘制。这里看到可以跟着滑动
		- 2、遍历所有的子view
		- 3、找到分组中的第一个view。在上方绘制宽分割线，其他的都绘制细分割线
- ## 3、重写OnDrawOver,绘制当前组的吸顶效果
  collapsed:: true
	- ## 思路
		- > 吸顶的位置固定的，需要通过onDrawOver绘制在上方。而跟随列表滑动的组分割线，是通过onDraw绘制的
		- 1、找到当前屏幕第一个可见的ItemVIew对应的position。判断第一个可见的下一个是否是另一组的开始
		- 2、如果是另一个组的第一个条目。则开始处理 下一组分割线顶上一组的效果
		  collapsed:: true
			- ```java
			      // 当第二个是组的头部的时候,开始第二个组，顶第一个组
			              boolean isGroupHeader = adapter.isGourpHeader(position + 1);
			  ```
		- 3、实际上上一组的绘制的区域随着手指上移逐渐变小
			- ```java
			   int bottom = Math.min(groupHeaderHeight, itemView.getBottom() - parent.getPaddingTop());
			  ```
		- 4、如果第一个可见item。下一个条目还是本组的，那么onDrawOver一直在顶上绘制当前吸顶效果的那个宽组
		- 5、第二组把第一组顶上去，切换时间的说明：
			- 当2中第二组往上顶第一组，其实是正在吸顶的绘制区域逐渐变小了。下一组的通过onDraw跟随滑动的逐渐上移。造成的顶上去的视觉、等到第一组的变小为0时。已经触发了isGroupHeader当前可见的下一个不是另一组了。于是用onDrawOver。绘制了新组区域了