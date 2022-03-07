- [动画实现](https://www.cnblogs.com/guanxinjing/p/13528962.html)
- 2个重叠的viewPager,实现联动
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
	  
	  public class LinkBannerLayout extends FrameLayout {
	      public BGAGuideLinkageLayout(Context context) {
	          super(context);
	      }
	  
	      public BGAGuideLinkageLayout(Context context, AttributeSet attrs) {
	          super(context, attrs);
	      }
	  
	      public BGAGuideLinkageLayout(Context context, AttributeSet attrs, int defStyleAttr) {
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
-