## 问题原因
	- 1、BottomSheetDialog的BottomSheetBehavior有滑动处理
	- 2、webview有滑动处理
- ## 现象
	- 上划webView滑动卡顿
- ## 解决思路
	- 方案1、webView的down事件请求父view不拦截他的事件
		- ```kotlin
		          _binding?.webView?.setOnTouchListener { view, ev ->
		              if (ev.action == MotionEvent.ACTION_DOWN) {
		                  (view as WebView).requestDisallowInterceptTouchEvent(true)
		              }
		              false
		          }
		  ```
	- 方案2、处理嵌套滑动 父子view滑动问题，借用nestScrollView
		-