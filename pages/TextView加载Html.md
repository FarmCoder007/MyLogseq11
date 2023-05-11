- 1、替换
	- 将指定的字 变色加粗
	- ```java
	  val text = "测试api:fun log(logHook: (log: String) -> String?)\n测试case:\nModuleLogger.d{\"春眠不觉晓\"}\nModuleLogger.d{\"处处闻啼鸟\"}\nhook 所有api新增参数:\"锄禾日当午\""
	  
	  val html = HtmlCompat.fromHtml(
	      text.replace("测试api", "<b><font color=\"#333333\">测试api</font></b>")
	          .replace("测试case", "<b><font color=\"#333333\">测试case</font></b>")
	          .replace("\n", "<br>")
	          .replace(":", "<font color=\"#999999\">:</font>"),
	      HtmlCompat.FROM_HTML_MODE_LEGACY
	  )
	  
	  textView.text = html
	  
	  ```