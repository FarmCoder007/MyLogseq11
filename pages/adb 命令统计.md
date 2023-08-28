- adb shell am start -S -W [packageName]/[activityName]
- ## 示例
	- ```java
	  adb [-d|-e|-s <serialNumber>] shell am start -S -W
	  com.example.app/.MainActivity
	  -c android.intent.category.LAUNCHER
	  -a android.intent.action.MAIN
	  ```
- ## 启动完成后，将输出：
	- ThisTime: 415
	  TotalTime: 415
	  WaitTime: 437
- WaitTime:总的耗时，包括前一个应用Activity pause的时间和新应用启动的时间；
  ThisTime表示一连串启动Activity的最后一个Activity的启动耗时；
  TotalTime表示新应用启动的耗时，包括新进程的启动和Activity的启动，但不包括前一个应用Activity pause
  的耗时。
  开发者一般只要关心TotalTime即可，这个时间才是自己应用真正启动的耗时。