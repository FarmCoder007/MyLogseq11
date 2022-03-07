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
	- ## viewPager
		-
	- ## viewPager2