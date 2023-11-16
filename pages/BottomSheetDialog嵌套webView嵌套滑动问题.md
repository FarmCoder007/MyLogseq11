## 问题原因
	- 1、BottomSheetDialog的BottomSheetBehavior有滑动处理
	- 2、webview有滑动处理
- ## 现象
	- 上划webView滑动卡顿
- ## 解决思路
	- 方案1、webView的down事件请求父view不拦截他的事件
		- ```kotlin
		  ```