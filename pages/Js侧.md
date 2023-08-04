- # 1、js声明各种方法代码
  collapsed:: true
	- ```kotlin
	  var xiangxuejs = {};
	  xiangxuejs.os = {};
	  xiangxuejs.os.isIOS = /iOS|iPhone|iPad|iPod/i.test(navigator.userAgent);
	  xiangxuejs.os.isAndroid = !xiangxuejs.os.isIOS;
	  // 存储所有回调方法
	  xiangxuejs.callbacks = {}
	  
	  /**
	   *  定义给Native调用的回调方法
	      安卓调用：evaluateJavascript("javascript:xiangxuejs.callback('" + callbackname + "'," + response + ")", null);
	      xiangxuejs.callback 为被调用函数名
	   */
	  xiangxuejs.callback = function (callbackname, response) {
	     // 收到Native调用 从数组按名字取真正的回调 去处理
	     var callbackobject = xiangxuejs.callbacks[callbackname];
	     console.log("xxxx"+callbackname);
	     if (callbackobject !== undefined){
	         if(callbackobject.callback != undefined){
	            console.log("xxxxxx"+response);
	              var ret = callbackobject.callback(response);
	             if(ret === false){
	                 return
	             }
	             delete xiangxuejs.callbacks[callbackname];
	         }
	     }
	  }
	  
	  /**
	   *  声明 js 调用 Native 方法  不带回调的
	      commandname 命令的名字 相当于action
	      parameters  通信的参数
	   */
	  xiangxuejs.takeNativeAction = function(commandname, parameters){
	      console.log("xiangxuejs takenativeaction")
	      // 将名字和参数 都放一个对象转json  传递给Native
	      var request = {};
	      request.name = commandname;
	      request.param = parameters;
	      if(window.xiangxuejs.os.isAndroid){
	          console.log("android take native action" + JSON.stringify(request));
	          // 安卓 拿到对象映射xiangxuewebview，调用Native方法
	          window.xiangxuewebview.takeNativeAction(JSON.stringify(request));
	      } else {
	          // ios 处理
	          window.webkit.messageHandlers.xiangxuewebview.postMessage(JSON.stringify(request))
	      }
	  }
	  
	  /**
	   *  声明 js 调用 Native 方法，带回调的
	   *  callback 这个是 js回调方法
	   */
	  xiangxuejs.takeNativeActionWithCallback = function(commandname, parameters, callback) {
	      // 定义回调名字，安卓来通过这个 调用js
	      var callbackname = "nativetojs_callback_" +  (new Date()).getTime() + "_" + Math.floor(Math.random() * 10000);
	      // 将所有回调 按生成的名字存起来
	      xiangxuejs.callbacks[callbackname] = {callback:callback};
	  
	      // 将回调名字 通过json 传给 Native
	      var request = {};
	      request.name = commandname;
	      request.param = parameters;
	      request.param.callbackname = callbackname;
	      if(window.xiangxuejs.os.isAndroid){
	          window.xiangxuewebview.takeNativeAction(JSON.stringify(request));
	      } else {
	          window.webkit.messageHandlers.xiangxuewebview.postMessage(JSON.stringify(request))
	      }
	  }
	  
	  window.xiangxuejs = xiangxuejs;
	  
	  ```
- # 2、Html代码
	- ```kotlin
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
	  <!--        定义多个方法-->
	          function callAppToast(){
	              console.log("callAppToast.");
	              xiangxuejs.takeNativeAction("showToast", {message: "this is a message from html."});
	          }
	          function openActivityA(){
	              console.log("openActivityA.");
	              xiangxuejs.takeNativeAction("openPage", {target_class: "com.xiangxue.webview.MainProcessTestActivity"});
	          }
	          function openActivityB(){
	              console.log("openActivityA.");
	              xiangxuejs.takeNativeAction("openPage", {target_class: "com.xiangxue.webview.MainProcessTestActivityB"});
	          }
	          function login(){
	              console.log("login.");
	              xiangxuejs.takeNativeActionWithCallback("login", {}, function(res) {
	                   var element = document.createElement("div");
	                   element.appendChild(document.createTextNode("userName: "+res.accountName));
	                   document.getElementById('login_id').appendChild(element);
	              });
	          }
	      </script>
	      <script src="js/xiangxuejs.js" charset="utf-8"></script>
	  </head>
	  <body style="height: 100%;">
	  <!--声明几个按钮，触发点击事件-->
	  <div class="item" style="font-size: 20px; color: #ffffff" onclick="callAppToast()">调用: showToast</div>
	  <div class="item" style="font-size: 20px; color: #ffffff" onclick="openActivityA()">打开MainProcessTestActivity</div>
	  <div class="item" style="font-size: 20px; color: #ffffff" onclick="openActivityB()">打开另外一个主进程的activity</div>
	  <div id="login_id" class="item" style="font-size: 20px; color: #ffffff" onclick="login()">登录并且返回结果</div>
	  
	  </body>
	  </html>
	  ```