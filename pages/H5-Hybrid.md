- ## 一、工作
  background-color:: #793e3e
  collapsed:: true
	- 1、注册配置及action-58ClientHybridLib
		- SampleConfig
			- actions中注册 action key 和 ctrl类
	- 2、初始化配置及action-WubaHybridSDK
		- MigrationCompact.init 注册action
		- HybridCtrlInjector注册action
- ## JSBridge
  background-color:: #793e3e
  collapsed:: true
	- ### 简介
	  collapsed:: true
		- JavaScript是运行在单独的JS Context中（Webview容器、JSCore等），与原生有运行环境的隔离，所以需要有一种机制实现Native端和Web端的双向通信，这就是JSBridge：以JavaScript引擎或Webview容器作为媒介，通过协定协议进行通信，实现Native端和Web端双向通信的一种机制。
		- 通过JSBridge，Web端可以调用Native端的Java接口，同样Native端也可以通过JSBridge调用Web端的JavaScript接口，实现彼此的双向调用
		- ![image.png](../assets/image_1665228399254_0.png)
	- ### 原理：实现JSBridge主要是两点：
		- 将Native端原生接口封装成JavaScript接口
		  将Web端JavaScript接口封装成原生接口
- ## 二、Android 与 JS交互传值
  background-color:: #793e3e
	- ### Android 4.4之前只能用loadUrl来实现，并且无法执行回调：
	  background-color:: #533e7d
	  collapsed:: true
		- ```java
		  Android:
		  webView.loadUrl("javascript:callJs('"0000"')");//需要JS function callJs函数
		  webView.loadUrl("http://172....&type=aaaa");//直接传递,无需JS function 
		  
		  JS:
		  function callJs(data){}
		  
		  ```
	- ### Android 4.4之后提供了evaluateJavascript来执行JS代码，并且可以获取返回值执行回调：
	  background-color:: #533e7d
		-
- ## 三、JS与Android交互
  collapsed:: true
	- ### 方式一：
		- ```java
		  JS:
		  function{
		  window.aile.callAndroid("0000000");
		  }
		  
		  Android:
		  // 注册模块
		  webView.addJavascriptInterface(this, "aile");
		  
		  /**
		   *  定义js接口方法
		   */
		  @JavascriptInterface
		  public void callAndroid(String src) {}
		  
		  ```
-
- ## 参考：
	- [混合开发总结](https://blog.csdn.net/ware00/article/details/110805684)
-
-