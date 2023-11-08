# 基础知识
collapsed:: true
	- ## 1、[[Android坐标系总结-面试]]
	- ## 2、[[getMeasuredWidth() 和 getWidth() 区别 和使用场景]]
	- ## 3、[[自定义View的基本方法]]
	- ## 4、自定义view和ViewGroup需要重新的方法#card
		- 自定义View: 只需要重写onMeasure()和onDraw()
		- 自定义ViewGroup: 则只需要重写onMeasure()和onLayout()，也可以重写onDraw绘制分割线
		- [[自定义view需要重写的方法-面试]]
	- ## 5、[[View构造函数+使用场景]]
	- ## 6、布局文件xml的键值对，在inflate函数解析的
	- ## 8、[[自定义view的绘制流程-生命周期方法]]
	- ## 4、[[onLayout和layout的区别？]]
	- ## 8、自定义view中为什么要Measure？#card
		- 为了给子view分配空间，需要结合==**自己layoutParams设置的大小**==，和[[#red]]==**父view规定的测量模式**==。去计算测量
	- ## 9、invalidate和 requestLayout区别？#card
		- invalidate布局没变化，刷新数据Ondraw
		- requestLayout,布局有变化，重新布局onMeasure onLayout onDraw
	- ## 10、刷新中Invalidate和postInvalidate的区别#card
		- invalidate只能在ui线程使用
		- postInvalidate可以在子线程中使用，内部也是用了handler
	- ## 11、 invalidate()和requestLayout()触发的生命周期#card
		- ![](https://img-blog.csdn.net/20160826104112634?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
- # 绘制文字
  collapsed:: true
	- ## 绘制文字，纵向起始位置
		- 纵向上设置的起始点位于文字的[[#red]]==**基线上 baseline**==、
		- 所以你drawText（“xxx”,0,0,paint）想从00 开始绘制时，实际baseline在 00 那条线，实际文字就在屏幕上方外部了
	- ## 9、[[自定义view-文字基线]]
	- ## 5、[[静态文字-纵向修正文字]]：getTextBounds获取文字绘制时的边界，找出偏移   -  偏移
	- ## 6、[[动态文字-纵向校正]]：(descent - accent)/2修正
- # 原理流程
  collapsed:: true
	- ## [[自定义viewGroup-面试题]]
	- ## 7、[[MeasureSpec-面试题]]
	- ## 3、[[view的布局流程（测量 布局）]]，绘制就是调用canvas api了
	- ## [[Activity布局流程/创建流程ActivityThread+PhoneWindow]]
	- ## [[View的绘制流程]]触发从setCopntetnView
	- ## 一个viewgroup的宽高是100dp,textview的宽高是wrapcontent。那么实际textview宽高多少？#card
		- 根据childMeasureSpec 可知子view是最大模式。最大不超过父容器的viewGroup 100dp
		- 那么 `TextView` 的实际宽度和高度将取决于其内容
		- ==**宽度**==： 如果 `TextView` 的内容足够小，它的宽度将适应内容的宽度，即 `wrap_content`。但如果内容太长，超过了 `ViewGroup` 的宽度（100dp），则 `TextView` 的宽度将被截断为 `ViewGroup` 的宽度。
		- ==**高度**==： `TextView` 的高度同样会根据内容的高度来动态调整，即 `wrap_content`。但是，如果内容过多，高度超过了 `ViewGroup` 的高度（100dp），那么 `TextView` 的高度将被限制为 `ViewGroup` 的高度。
	- ## 如果自定义view在xml里写的wrapcontent，那么怎么让他有个初始尺寸？#card
		- 在构造函数中，你可以设置 View 的默认宽高，然后在测量阶段进行适当的调整。
		- ```java
		  import android.content.Context;
		  import android.util.AttributeSet;
		  import android.view.View;
		  
		  public class CustomView extends View {
		  
		      // 默认宽高
		      private int defaultWidth = 200;
		      private int defaultHeight = 100;
		  
		      public CustomView(Context context) {
		          super(context);
		          init();
		      }
		  
		      public CustomView(Context context, AttributeSet attrs) {
		          super(context, attrs);
		          init();
		      }
		  
		      public CustomView(Context context, AttributeSet attrs, int defStyleAttr) {
		          super(context, attrs, defStyleAttr);
		          init();
		      }
		  
		      private void init() {
		          // 设置默认宽高
		          setMinimumWidth(defaultWidth);
		          setMinimumHeight(defaultHeight);
		      }
		  
		      @Override
		      protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
		          // 处理 wrap_content 的情况
		          int width = resolveSize(defaultWidth, widthMeasureSpec);
		          int height = resolveSize(defaultHeight, heightMeasureSpec);
		          setMeasuredDimension(width, height);
		      }
		  }
		  
		  ```
- # 嵌套滑动-解决问题：父子联动#card
	- ## 1、淘宝嵌套吸顶解决方法
		- 吸顶问题方案：将tabLayout+RecyclerView 算成ScrollView的最后一个子view。且合起来高度 = 屏幕高度
	- ## 2、嵌套滑动:NestedScrollingChild、NestedScrollingParent配合
		- ### 思路：滑动开始时，先看父view还可以滑动吗，不可以滑动，子view再滑动
		- ## 流程
			- 1、嵌套滑动主动者、触发者是子view，所以子view先得开启支持嵌套滑动setNestedScrollingEnabled = true （recyclerView 默认开启）
			- >2、然后手指按下onTouchevent的==**Action_Down事件**==触发嵌套滑动回调
			- 3、子view走开始滑动的回调[[#red]]==**startNestedScroll**==：逐层向上找到第一个支持嵌套滑动的父view。
				- 它会逐层调用父view的[[#red]]==**onStartNestedScroll**==询问是否支持嵌套滑动，如果父view支持嵌套滑动的话会调用[[#red]]==**onNestedScrollAccepted**==，作为标识支持
			- > 4、Action_move事件，开始父子联动配合，父view能滑动就先滑，滑不动再子view滑
			- 5、[[#red]]==**重点**==子view滑动前，调用[[#green]]==**dispatchNestedPreScroll**==，先问父亲能不能滑
			- 6、父view回调[[#green]]==**onNestedPreScroll**==。根据业务需求去处理自己能不能滑动，
				- 比如判断吸顶了，父view就不滑动了，不吸顶就滑动。滑动了需要将滑动距离赋值到consumed参数标识消费了滑动距离。省的子view在父view滑动时也滑动
	- ## 参考
	  collapsed:: true
		- ## [[模仿淘宝首页滑动]]
		- [[嵌套滑动父子view联动]]
- # 滑动冲突-解决问题：内部拦截外部拦截
  collapsed:: true
	- ## [[滑动事件冲突]]
	- 比如ViewPager嵌套ListView。不让系统处理冲突。自己实现的话解决思路
	- ## 法1、内部拦截法：ListView自己处理冲突
		- ## 1、自定义ListView,重写 dispatchTouchEvent方法
			- 1、down事件请求父view不拦截：getParent().requestDisallowInterceptTouchEvent(true);
			- 2、move事件判断横向滑动 请求父view viewPager 拦截  getParent().requestDisallowInterceptTouchEvent(false);
			- viewgroup的源码。disPacthTouchEvent里。Down事件会清除标记。上边请求父view不拦截的标记 会被清除那么 viewPager也要做下处理
			- 代码
			  collapsed:: true
				- ```java
				  public class MyListView extends ListView {
				  
				      public MyListView(Context context) {
				          super(context);
				      }
				  
				      public MyListView(Context context, AttributeSet attrs) {
				          super(context, attrs);
				      }
				  
				      // 内部拦截法：子view处理事件冲突
				      private int mLastX, mLastY;
				  
				      @Override
				      public boolean dispatchTouchEvent(MotionEvent event) {
				          int x = (int) event.getX();
				          int y = (int) event.getY();
				  
				          switch (event.getAction()) {
				              // down事件请求父view不拦截
				              case MotionEvent.ACTION_DOWN: {
				                  getParent().requestDisallowInterceptTouchEvent(true);
				                  break;
				              }
				              case MotionEvent.ACTION_MOVE: {
				                  int deltaX = x - mLastX;
				                  int deltaY = y - mLastY;
				                  if (Math.abs(deltaX) > Math.abs(deltaY)) {
				                      // move事件判断横向滑动 运行 viewPager 拦截
				                      getParent().requestDisallowInterceptTouchEvent(false);
				                  }
				                  break;
				              }
				              case MotionEvent.ACTION_UP: {
				                  break;
				  
				              }
				              default:
				                  break;
				          }
				  
				          mLastX = x;
				          mLastY = y;
				          return super.dispatchTouchEvent(event);
				      }
				  }
				  ```
		- ## 2、ViewPager，需设置onInterceptTouchEvent DOWN事件不拦截就是return false
	- ## 法2、外部拦截法：：父容器处理冲突，我想要把事件分发给谁就分发给谁
		- ListView不处理
		- 自定义ViewPager，重写onInterceptTouchEvent拦截方法。判断Move事件 横向滑动才拦截，其他不拦截
- # 怎么实现图片圆角#card
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