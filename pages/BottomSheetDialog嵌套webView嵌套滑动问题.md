## 上划卡顿问题
collapsed:: true
	- ## 原因
		- 1、BottomSheetDialog的BottomSheetBehavior有滑动处理
		- 2、webview有滑动处理
	- ## 现象
		- 上划webView滑动卡顿
	- ## 解决思路
	  collapsed:: true
		- 方案1、webView的down事件请求父view不拦截他的事件
			- java
				- ```java
				  webView.setOnTouchListener(new View.OnTouchListener() {
				      @Override
				      public boolean onTouch(View v, MotionEvent event) {
				          v.getParent().requestDisallowInterceptTouchEvent(true);
				          return false;
				      }
				  });
				  
				  ```
			- kotlin
				- ```kotlin
				          _binding?.webView?.setOnTouchListener { view, ev ->
				              if (ev.action == MotionEvent.ACTION_DOWN) {
				                  (view as WebView).requestDisallowInterceptTouchEvent(true)
				              }
				              false
				          }
				  ```
		- 方案2、处理嵌套滑动 父子view滑动问题，**使用 NestedScrollView 包装 WebView:**
			- ```xml
			  <androidx.core.widget.NestedScrollView
			      xmlns:android="http://schemas.android.com/apk/res/android"
			      android:layout_width="match_parent"
			      android:layout_height="match_parent">
			  
			      <WebView
			          android:id="@+id/myWebView"
			          android:layout_width="match_parent"
			          android:layout_height="wrap_content"/>
			  </androidx.core.widget.NestedScrollView>
			  ```
-