- 1、查看当前activity
	- adb shell dumpsys activity top | grep ACTIVITY
	- adb shell dumpsys activity activities
- 2、clear app 清除缓存
	- adb shell pm clear com.wuba
- 3、[adb logcat 命令](https://blog.csdn.net/zhaohuih/article/details/106905219)
-
- 4、
- # mac zsh下配置adb
  collapsed:: true
	- 1、**添加路径到 PATH 环境变量**：
		- 1、打开shell配置
			- ```
			  open -e .zshrc
			  ```
		- 2、确保 Android SDK 的 `platform-tools` 目录被添加到 PATH 中。例如：注意/path/to/android-sdk/platform-tools替换成自己android-sdk的路径
			- ```
			  export PATH="$PATH:/path/to/android-sdk/platform-tools"
			  ```
		- 3、重启source ~/.zshrc