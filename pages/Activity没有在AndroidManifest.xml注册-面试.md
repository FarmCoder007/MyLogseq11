title:: Activity没有在AndroidManifest.xml注册-面试

- # 1、错误在哪里检测的？
  collapsed:: true
	- 1、启动activity的时候会执行到execStartActivity里，然后执行AMS的startActivity方法
	- 2、检测上面返回结果checkStartActivityResult
		- ActivityManager.START_INTENT_NOT_RESOLVED:
		- ActivityManager.START_CLASS_NOT_FOUND:
		- 是这2个返回结果就会报错。
- # 2、ActivityStarter.startActivity[2]中返回了START_INTENT_NOT_RESOLVED，START_CLASS_NOT_FOUND
  collapsed:: true
	- ```java
	        //接下来开始做一些校验判断
	        if (err == ActivityManager.START_SUCCESS && intent.getComponent() == null) {
	            // We couldn't find a class that can handle the given Intent.
	            // That's the end of that!
	            //从Intent中无法找到相应的Component
	            err = ActivityManager.START_INTENT_NOT_RESOLVED;
	        }
	        if (err == ActivityManager.START_SUCCESS && aInfo == null) {
	            // We couldn't find the specific class specified in the Intent.
	            // Also the end of the line.
	            //从Intent中无法找到相应的ActivityInfo
	            err = ActivityManager.START_CLASS_NOT_FOUND;
	        }
	  ```
- # 3、intent.getComponent()和aInfo（ActivityInfo）赋值的地方
	- intent.getComponent()来源于 startActivityMayWait
	- activityinfo来源于 ResolveInfo（在getActivityInfo赋值的）
- 学完pkms还可以再解答 清单文件哪里解析的