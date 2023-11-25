# 一、基础
	- ## [基础知识](https://mp.weixin.qq.com/s/5Hs9Bg93IbY2uRUMeIqAJQ)
	  collapsed:: true
		- ![image.png](../assets/image_1690948949406_0.png)
		- ![image.png](../assets/image_1690898812840_0.png)
	- ## JavascriptInterface:通信用
	- ## 打印html的日志到控制台
	  collapsed:: true
		- ![image.png](../assets/image_1690959521134_0.png)
	- ### WebViewClient的方法[shouldOverrideUrlLoading](https://developer.android.com/reference/android/webkit/WebViewClient#shouldOverrideUrlLoading(android.webkit.WebView,%20java.lang.String))
		- 返回 `true`会导致当前 WebView 中止加载 URL
		- 而返回会 `false`导致 WebView 像往常一样继续加载 URL。
- # 二、[[Webview和Native之间的通信]]
- # 三、[[webview放入独立的进程上]]
  collapsed:: true
	- ![image.png](../assets/image_1690958411781_0.png)
- # 四、禁用webView的触摸事件
  collapsed:: true
	- ### 背景
		- viewPager2 嵌套 webView 需要viewPager2响应点击事件
	- ### 修改
		- ```kotlin
		  class MyWebView: WebView {
		      constructor(context: Context) : super(context)
		  
		      constructor(context: Context, attrs: AttributeSet) : super(context, attrs)
		  
		      constructor(context: Context, attrs: AttributeSet, defStyleAttr: Int) : super(context, attrs, defStyleAttr)
		  
		      override fun onTouchEvent(event: MotionEvent?): Boolean {
		          return false
		      }
		  }
		  ```
- # 五、webView设置背景透明去掉白色
  collapsed:: true
	- ```java
	  // 设置背景色 
	  mWebView.setBackgroundColor(0); 
	  // 设置填充透明度
	  mWebView.getBackground().setAlpha(0)
	  ```
- # 四、高阶+优化
  collapsed:: true
	- [[解决WebView多进程崩溃]]
	- [[Webview优化]]
- # 面试题
	- [[WebView面试题]]
-