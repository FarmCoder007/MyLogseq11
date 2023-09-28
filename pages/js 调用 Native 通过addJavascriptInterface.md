# Native端
	- ## 1、addJavascriptInterface添加对象映射
		- js使用的时候先通过Window获取xiangxuewebview，拿到对象 才能调方法
		- ```kotlin
		      public void init() {
		          WebViewProcessCommandDispatcher.getInstance().initAidlConnection();
		          WebViewDefaultSettings.getInstance().setSettings(this);
		          // 参数1 映射对象  参数2 接口名
		          addJavascriptInterface(this, "xiangxuewebview");
		      }
		  
		  ```
	- ## 2、 通过@JavascriptInterface 声明可调的方法
		- js拿到对象映射可以调这个方法。调用takeNativeAction这个方法
		- ```kotlin
		      // 定义一个参数，ios侧只能接受一个参数 ，通过json再解析
		     @JavascriptInterface
		      public void takeNativeAction(final String jsParam) {
		          Log.i(TAG, jsParam);
		          if (!TextUtils.isEmpty(jsParam)) {
		              final JsParam jsParamObject = new Gson().fromJson(jsParam, JsParam.class);
		              if (jsParamObject != null) {
		                // Native 收到 js 传过来的json 去解析对应的action 命令， 根据命令去分发   
		                WebViewProcessCommandDispatcher.getInstance().executeCommand(jsParamObject.name, new Gson().toJson(jsParamObject.param), this);
		              }
		          }
		      }
		  ```
- # JS
	- ## 1、js声明方法 ，根据不同客户端去获取映射对象，调用Native
		- ```kotlin
		  /**
		   *  声明 js 调用 Native 方法，不带回调的
		      参数1 命令的名字
		      参数2 参数
		   */
		  xiangxuejs.takeNativeAction = function(commandname, parameters){
		      console.log("xiangxuejs takenativeaction")
		      // 将名字和参数 都放一个对象转json  传递给Native
		      var request = {};
		      request.name = commandname;
		      request.param = parameters;
		      if(window.xiangxuejs.os.isAndroid){
		          console.log("android take native action" + JSON.stringify(request));
		          // 安卓 拿到对象映射，调用Native方法
		          window.xiangxuewebview.takeNativeAction(JSON.stringify(request));
		      } else {
		          // ios 处理
		          window.webkit.messageHandlers.xiangxuewebview.postMessage(JSON.stringify(request))
		      }
		  }
		  ```
	- ## 2、js方法应用到Html中处理点击事件，调用js代码 去获取Native功能
		- ```html
		  <!DOCTYPE html>
		  <html lang="en">
		  <head>
		      <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		      <meta name="format-detection" content="telephone = no">
		      <meta name="viewport"
		            content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=no">
		      <meta name="apple-mobile-web-app-capable" content="yes">
		      <style type="text/css">
		           .item {
		           padding: 20px;
		           max-width: 600px;
		           margin: 0 auto;
		           text-align: center;
		           background-color: #999999;
		           margin-top: 20px;
		           }
		        </style>
		      <script>
		          // 定义方法
		          function callAppToast(){
		              console.log("callAppToast.");
		              xiangxuejs.takeNativeAction("showToast", {message: "this is a message from html."});
		          }
		  
		      </script>
		      <script src="js/xiangxuejs.js" charset="utf-8"></script>
		  </head>
		  <body style="height: 100%;">
		    // 点击触发
		  <div class="item" style="font-size: 20px; color: #ffffff" onclick="callAppToast()">调用: showToast</div>
		  
		  </body>
		  </html>
		  ```
	- # 总结
		- 主要是这行
		- ```kotlin
		  
		  window.xiangxuewebview.takeNativeAction(JSON.stringify(request));
		  ```
		- xiangxuewebview:native 添加映射对象时，名字
		- takeNativeAction：Native 被@JavascriptInterface修饰的方法，可以被调用