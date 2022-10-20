- # 一、[动画实现](https://www.cnblogs.com/guanxinjing/p/13528962.html)
- # 二、两个重叠的viewPager,实现联动引导页
  collapsed:: true
	- xml:
		- ```
		  <com.wuba.a3dbanner.LinkBannerLayout xmlns:android="http://schemas.android.com/apk/res/android"
		      android:id="@+id/guideLinkageLayout"
		      xmlns:app="http://schemas.android.com/apk/res-auto"
		      xmlns:tools="http://schemas.android.com/tools"
		      android:layout_width="match_parent"
		      android:layout_height="200dp"
		      tools:context=".MainActivity">
		  
		      <androidx.viewpager2.widget.ViewPager2
		          android:id="@+id/banner_bg"
		          android:layout_width="match_parent"
		          android:layout_height="match_parent"
		          />
		      <androidx.viewpager2.widget.ViewPager2
		          android:id="@+id/banner_fg"
		          android:layout_width="match_parent"
		          android:layout_height="match_parent"
		          />
		  
		  </com.wuba.a3dbanner.LinkBannerLayout>
		  ```
	-
	- LinkBannerLayout:
	- ```
	  /**
	   *  描述:联动布局，将每一个触摸事件分发给所有的子控件。
	   */
	  public class LinkBannerLayout extends FrameLayout {
	      public LinkBannerLayout(Context context) {
	          super(context);
	      }
	  
	      public LinkBannerLayout(Context context, AttributeSet attrs) {
	          super(context, attrs);
	      }
	  
	      public LinkBannerLayout(Context context, AttributeSet attrs, int defStyleAttr) {
	          super(context, attrs, defStyleAttr);
	      }
	  
	      @Override
	      public boolean dispatchTouchEvent(MotionEvent ev) {
	          for (int i = 0; i < getChildCount(); i++) {
	              View child = getChildAt(i);
	              try {
	                  child.dispatchTouchEvent(ev);
	              } catch (Exception e) {
	                  e.printStackTrace();
	              }
	          }
	          return true;
	      }
	  }
	  
	  ```
- # 三、切换动画，淡入淡出
  collapsed:: true
	- ```
	  
	  bgBanner.setPageTransformer(NGGuidePageTransformer())
	  class NGGuidePageTransformer : BasePageTransformer() {
	          override fun transformPage(view: View, position: Float) {
	              val pageWidth = view.width //得到view宽
	              if (position < -1) { // [-Infinity,-1)
	                  // This page is way off-screen to the left. 出了左边屏幕
	                  view.alpha = 0f
	              } else if (position <= 1) { // [-1,1]
	                  if (position < 0) {
	                      //消失的页面
	                      view.translationX = -pageWidth * position //阻止消失页面的滑动
	                  } else {
	                      //出现的页面
	                      view.translationX = pageWidth.toFloat() //直接设置出现的页面到底
	                      view.translationX = -pageWidth * position //阻止出现页面的滑动
	                  }
	                  // Fade the page relative to its size.
	                  val alphaFactor = Math.max(MIN_ALPHA, 1 - Math.abs(position))
	                  //透明度改变Log
	                  view.alpha = alphaFactor
	              } else { // (1,+Infinity]
	                  // This page is way off-screen to the right.    出了右边屏幕
	                  view.alpha = 0f
	              }
	          }
	  
	          companion object {
	              private const val MIN_ALPHA = 0.0f //最小透明度
	          }
	      }
	  ```
- # 四、控制切换page的速度
  collapsed:: true
	- ## viewPager
		- ### 1、自定义BannerScroller 调整设置调用setCurrentItem(int item, boolean smoothScroll)方法时，page切换的时间长度
		  collapsed:: true
			- ```
			  public class BannerScroller extends Scroller {
			      private int mDuration = 1000;
			  
			      public BGABannerScroller(Context context, int duration) {
			          super(context);
			          mDuration = duration;
			      }
			  
			      @Override
			      public void startScroll(int startX, int startY, int dx, int dy) {
			          super.startScroll(startX, startY, dx, dy, mDuration);
			      }
			  
			      @Override
			      public void startScroll(int startX, int startY, int dx, int dy, int duration) {
			          super.startScroll(startX, startY, dx, dy, mDuration);
			      }
			  }
			  ```
		- ### 2、自定义viewPager ,扩展方法反射设置 scroller
		  collapsed:: true
			- ```
			  public void setPageChangeDuration(int duration) {
			          try {
			              Field scrollerField = ViewPager.class.getDeclaredField("mScroller");
			              scrollerField.setAccessible(true);
			              scrollerField.set(this, new BGABannerScroller(getContext(), duration));
			          } catch (Exception e) {
			              e.printStackTrace();
			          }
			      }
			  ```
	- ## viewPager2
		- ### 1、不能通过反射获取”mscroller“
			- java:
				- ```
				  public class MyPagerHelper {
				      /**
				       * 保存前一个animatedValue
				       */
				      private static int previousValue;
				  
				      /**
				       * 设置当前Item
				       * @param pager    viewpager2
				       * @param item     下一个跳转的item
				       * @param duration scroll时长
				       */
				      public static void setCurrentItem(final ViewPager2 pager, int item, long duration) {
				          previousValue = 0;
				          int currentItem = pager.getCurrentItem();
				          int pagePxWidth = pager.getWidth();
				          int pxToDrag = pagePxWidth * (item - currentItem);
				          final ValueAnimator animator = ValueAnimator.ofInt(0, pxToDrag);
				          animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
				              @Override
				              public void onAnimationUpdate(ValueAnimator animation) {
				                  int currentValue = (int) animation.getAnimatedValue();
				                  float currentPxToDrag = (float) (currentValue - previousValue);
				                  pager.fakeDragBy(-currentPxToDrag);
				                  previousValue = currentValue;
				              }
				          });
				          animator.addListener(new Animator.AnimatorListener() {
				              @Override
				              public void onAnimationStart(Animator animation) {
				                  pager.beginFakeDrag();
				              }
				  
				              @Override
				              public void onAnimationEnd(Animator animation) {
				                  pager.endFakeDrag();
				              }
				  
				              @Override
				              public void onAnimationCancel(Animator animation) { }
				  
				              @Override
				              public void onAnimationRepeat(Animator animation) { }
				          });
				          animator.setInterpolator(new AccelerateDecelerateInterpolator());
				          animator.setDuration(duration);
				          animator.start();
				      }
				  }
				  
				  
				  使用： MyPagerHelper.setCurrentItem(foregroundBanner,currentPosition,500)
				  
				  ```
				-
			- kotlin:
			  collapsed:: true
				- ```
				  fun ViewPager2.setCurrentItem(
				          item: Int,
				          duration: Long,
				          interpolator: TimeInterpolator = AccelerateDecelerateInterpolator(),
				          pagePxWidth: Int = width // 使用viewpager2.getWidth()获取
				      ) {
				          val pxToDrag: Int = pagePxWidth * (item - currentItem)
				          val animator = ValueAnimator.ofInt(0, pxToDrag)
				          var previousValue = 0
				          animator.addUpdateListener { valueAnimator ->
				              val currentValue = valueAnimator.animatedValue as Int
				              val currentPxToDrag = (currentValue - previousValue).toFloat()
				              fakeDragBy(-currentPxToDrag)
				              previousValue = currentValue
				          }
				          animator.addListener(object : Animator.AnimatorListener {
				              override fun onAnimationStart(animation: Animator?) { beginFakeDrag() }
				              override fun onAnimationEnd(animation: Animator?) { endFakeDrag() }
				              override fun onAnimationCancel(animation: Animator?) { }
				              override fun onAnimationRepeat(animation: Animator?) { }
				          })
				          animator.interpolator = interpolator
				          animator.duration = duration
				          animator.start()
				      }
				      
				      使用：直接用viewpager2 实例调
				  ```
		-
		-
	-
- # 五、禁止滑动,去除滑动动画
  collapsed:: true
	- 1、ViewPager2禁止滑动
	  调用setUserInputEnabled方法，设置是否可以手动滑动
	- 2、ViewPager2去除滑动效果
		- viewPager2.setCurrentItem(position,false)
	-
	-
- # 六、层叠切换动画
	- [原理](https://www.cnblogs.com/lzh-Linux/p/9001235.html)
	  [代码](https://blog.csdn.net/Jeffray1991/article/details/116196193)
	- 自己仓库里有代码
- # 七、画廊效果
	- https://www.jianshu.com/p/74830b692933
	- ![image.png](../assets/image_1666232206400_0.png)
	- # 关键点：
		- ## 7-1、让前中后页同时展示出来
			- ```xml
			  <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
			      android:layout_width="match_parent"
			      android:layout_height="match_parent"
			      android:clipChildren="false">
			  
			      <com.proxy.bannertest.view.LoopViewPager
			          android:id="@+id/myviewpager"
			          android:layout_width="match_parent"
			          android:layout_height="125dp"
			          android:layout_marginHorizontal="30dp"
			          android:clipChildren="false"/>
			      <LinearLayout
			          android:id="@+id/layout_dots"
			          android:layout_width="wrap_content"
			          android:layout_height="30dp"
			          android:layout_alignParentBottom="true"
			          android:layout_centerHorizontal="true"
			          android:gravity="center"
			          android:orientation="horizontal" />
			  
			  </RelativeLayout>
			  ```
- # 开源库
	- [banner](https://github.com/youth5201314/banner)
	- [viewPager上下联动](https://www.jianshu.com/p/a9518ec62640)