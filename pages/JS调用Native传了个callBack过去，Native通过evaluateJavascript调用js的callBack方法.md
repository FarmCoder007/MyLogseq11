- 假设第一步收到了callbackName为：xiangxuejs.callback
- # Native
	- 1、通过webView执行 evaluateJavascript("javascript:xiangxuejs.callback('" + 参数1+ "'," + 参数2+ ")", null);
- # JS，需要声明这个方法,内容是js内容，重点看方法
	- ```html
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
	  ```