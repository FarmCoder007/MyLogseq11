# 概念
	- 虽然您可能只需将“最小宽度”和“可用宽度”限定符结合使用即可支持所有尺寸变化，但是您可能还希望当
	  用户在纵向与横向之间切换屏幕方向时改变用户体验。
	  为此，您可以将 port 或 land 限定符添加到资源目录名称中。只需确保这些限定符在其他尺寸限定符
	  后面即可。例如：
- # 使用
	- ```java
	  res/layout/main_activity.xml # For handsets
	  res/layout-land/main_activity.xml # For handsets in landscape
	  res/layout-sw600dp/main_activity.xml # For 7” tablets
	  res/layout-sw600dp-land/main_activity.xml # For 7” tablets in landscape
	  ```
- 如需详细了解所有屏幕配置限定符，[请参阅提供资源指南中的表 2](https://developer.android.com/guide/topics/resources/providing-resources?hl=zh-cn)