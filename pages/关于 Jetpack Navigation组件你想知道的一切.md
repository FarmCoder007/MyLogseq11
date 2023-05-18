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
  collapsed:: true
	- Navigation 组件是一组用于简化 Android 导航的库、插件和工具。 Android Jetpack 的 Navigation 组件帮助我们实现导航，从简单的按钮点击到更复杂的模式，例如应用栏和导航抽屉。 导航组件还通过遵守一套既定的原则来确保一致且可预测的用户体验。
	- 导航组件由三个关键部分组成。
	- ## Navigation graph (导航图)
	  collapsed:: true
		- 这是一种新的资源类型——包含所有与导航相关的信息的 XML 文件。包括应用程序中所有单独的内容区域（称为目的地），以及应用中可以导航的路径。
		  collapsed:: true
			- ![image.png](../assets/image_1684414666466_0.png){:height 675, :width 716}
		- 导航编辑器中的 Navgraph 可以如上所示进行可视化编辑。上面的每一个 Fragment 被称为 目的地，这些 目的地 之间的箭头称之为 操作 —— 定义了用户可以导航的路径。
	- ## NavHost
	  collapsed:: true
		- NavHost 是一个空的容器，用于显示导航图中的目的地。 Navigation 组件包含一个默认的 NavHost 实现：NavHostFragment，它显示片段目的地。
		-
		-
		-
		-
	- ## NavController
	  collapsed:: true
		- NavController 是通过 NavHost 来管理应用中的导航。当用户在应用中执行页面导航时，NavController 会协调 NavHost 中目标内容的交换。
		- 注意：Android Studio 3.3 提供了导航编辑器中用来展示导航图。这个很棒的功能让我们可以在一个地方看到所有的导航。
- # 好处
	- 导航组件提供了许多其他好处，包括：
		- 处理 Fragment 事务。
		- 默认情况下正确处理 Up 和 Back 操作。
		- 为动画和过渡提供标准化资源。
		- 实现和处理深度链接。
		- 包括导航类型 UI 模式，例如只需最少的额外工作即可处理导航抽屉和底部导航。
		- Safe Args — 一个 Gradle 插件，在目的地之间导航和传递数据时提供类型安全。
		- 支持 ViewModel —— 实现在不同的目的地之间共享 UI 相关的数据
- # 示例
	- 让我们通过创建一个简单的例子来看看 Navigation 组件是如何工作的。 接下来将创建一个带有两个 Fragment 的简单 Activity 来看看如何使用 Navigation 组件实现
	  Fragment 之间的导航。
	-