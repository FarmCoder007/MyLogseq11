- [Everything You Want to Know About Android Jetpack’s Navigation Component](https://betterprogramming.pub/everything-about-android-jetpacks-navigation-component-b550017c7354)
- ![image.png](../assets/image_1684414584249_0.png)
- 在 Android 中，我们通常使用 Intent 实现 Activity 的导航逻辑，使用 fragment transactions 实现 Fragment 的导航逻辑。 Google 的 Navigation 导航组件简化了 Android 应用程序中的导航逻辑。
- 在这篇文章中，我们将讨论 Navigation 组件的基本使用方法和高级使用方法。
- # 过去有什么问题？
	- 在开发具有多个 Fragment 的应用程序时，我们倾向于执行大量 fragment transactions 以实现相互之间的跳转。编写这些片段事务并处理返回堆栈时，一个问题是处理成本比较高，另一个问题是是如果处理不完善，会导致 IllegalStateException 异常。
- # 解决方案
	- 为了让开发导航的成本更简单，Google 引入了 Navigation 组件。使用这个导航组件，可以轻松编写 Fragment 之间的导航以及处理返回堆栈、异常情况等情况。
	- 让我们开始探索 Navigation 组件。
- # 什么是 Navigation 组件？
	- Navigation 组件是一组用于简化 Android 导航的库、插件和工具。 Android Jetpack 的 Navigation 组件帮助我们实现导航，从简单的按钮点击到更复杂的模式，例如应用栏和导航抽屉。 导航组件还通过遵守一套既定的原则来确保一致且可预测的用户体验。
	- 导航组件由三个关键部分组成。
	- ## Navigation graph (导航图)
	  collapsed:: true
		- 这是一种新的资源类型——包含所有与导航相关的信息的 XML 文件。包括应用程序中所有单独的内容区域（称为目的地），以及应用中可以导航的路径。
		  collapsed:: true
			- ![image.png](../assets/image_1684414666466_0.png){:height 675, :width 716}
		- 导航编辑器中的 Navgraph 可以如上所示进行可视化编辑。上面的每一个 Fragment 被称为 目的地，这些 目的地 之间的箭头称之为 操作 —— 定义了用户可以导航的路径。
	- ## NavHost
	-