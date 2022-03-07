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
		  
		      <com.youth.banner.indicator.RectangleIndicator
		          android:id="@+id/indicator"
		          android:layout_width="wrap_content"
		          android:layout_height="20dp"
		          android:layout_margin="10dp"
		          android:layout_gravity="bottom"
		          />
		      <androidx.viewpager2.widget.ViewPager2
		          android:id="@+id/banner"
		          android:layout_width="match_parent"
		          android:layout_height="match_parent"
		          />
		  
		  </com.wuba.a3dbanner.LinkBannerLayout>
		  ```
	-
	- ```
	  ```
-