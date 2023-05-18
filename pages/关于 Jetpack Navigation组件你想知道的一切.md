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
	- ## 第1步
		- 创建一个基于 AndroidX 的新项目。AndroidX 是 Android 团队用于在 Jetpack 中开发、测试、打包、版本和发布库的开源项目。可以在 AndroidX 概览 中查看更多信息。
	- ## 第2步
	  collapsed:: true
		- 在 build.gradle 中添加依赖
		  collapsed:: true
			- ```
			  dependencies {
			    def nav_version = "2.3.0-alpha02"
			  
			    // Java language implementation
			    implementation "androidx.navigation:navigation-fragment:$nav_version"
			    implementation "androidx.navigation:navigation-ui:$nav_version"
			  
			    // Kotlin
			    implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
			    implementation "androidx.navigation:navigation-ui-ktx:$nav_version"
			  
			    // Dynamic Feature Module Support
			    implementation "androidx.navigation:navigation-dynamic-features-fragment:$nav_version"
			  
			    // Testing Navigation
			    androidTestImplementation "androidx.navigation:navigation-testing:$nav_version"
			  }
			  ```
		- 这些是针对不同需求的不同依赖项。根据需求进行选择。
	- ## 第3步 - 1
	  collapsed:: true
		- 创建导航图。
		- 要将导航图添加到您的项目：
		  collapsed:: true
			- 右键单击 res 目录并选择 New > Android Resource File，出现 New Resource File 对话框
			- 输入文件名，例如：nav_graph
			- 在 Resource type 的下拉列表中选择 Navigation，然后单击确定
				- ![image.png](../assets/image_1684414877484_0.png)
		- 当添加第一个导航图时，Android Studio 会在 res 目录中创建一个导航资源文件夹，该文件夹中包含导航图资源文件。创建的文件看起来像这样：
		  collapsed:: true
			- ```
			  <?xml version="1.0" encoding="utf-8"?>
			  <navigation xmlns:android="http://schemas.android.com/apk/res/android"
			      xmlns:app="http://schemas.android.com/apk/res-auto"
			      android:id="@+id/nav_graph">
			  </navigation>
			  ```
		- <navigation> 元素是导航图的根元素。当向图表添加目的地和连接操作时，会添加相应的 <destination> 和 <action> 元素作为子元素。如果有嵌套的图形，将显示为子 <navigation> 元素。
	- ## 第3步 - 2
	  collapsed:: true
		- 添加 NavHost 到 Activity 的 XML 文件中。
		- nav_host_fragment 中包含：
			- android:name：NavHost 的类名。
			- app:navGraph：将 NavHostFragment 与导航图相关联。导航图指定了此 NavHostFragment 中用户可以导航到的所有目的地。
			- app:defaultNavHost="true"：确保 NavHostFragment 拦截系统后退按钮。 请注意，只有一个NavHost 可以是默认值。 如果在同一布局中有多个 NavHost（例如双窗格布局），请确保仅指定一个默认 NavHost。
	- ## 第4步
		- 在 nav_graph 中添加目的地和路径。
		  在添加目的地之前创建两个 Fragment 及其 XML：
		- ![image.png](../assets/image_1684414981596_0.png)