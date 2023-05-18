- # 前言
  collapsed:: true
	- 跨平台的开发技术一直是热门的技术方向，不论是为了追求应用逻辑和用户体验的一致性，还是为了提升研发效率以及降低成本，各大互联网公司都在投入跨平台开发方案的研发和使用。Android 和 iOS 是移动端的两大主流系统平台，但它们都有各自完善而独立的技术生态系统，针对这两个平台的跨平台技术方案层出不穷，比如 H5、RN、小程序和 Flutter，以及各公司运用这些技术进行二次开发包装成的跨平台框架。
	- 作为移动端应用开发者，我们在日常开发中对以下问题深有体会：
	- 基础功能模块逻辑相对复杂，多端研发成本较高
	  双端逻辑代码膨胀，无法保证完全一致，问题排查难
	  逻辑代码穿插在各个模块中，现有跨端复用方案迁移成本高
	  尤其在移动互联网应用进入存量时代，App 不再追求双端各自快速实现功能，而是确保双端功能稳定性和逻辑一致性。而随着不断迭代，很多逻辑写着写着双端就出现不一样的细节了，出了问题不好查，而且往往 BI 反馈的双端数据表现不一致，维护迭代后又产生新的差异，如果再出现人员变动，那一部分代码就会变成「祖传代码」。
	- 那么，如果将应用的逻辑层代码剥离成公共模块供双端复用，是否可以改善上面的问题呢？
- # 初识KMM，出生名门
  collapsed:: true
	- KMM 的全称是 Kotlin Multiplatform Mobile，它是大名鼎鼎的 JetBrains 开发的基于 Kotlin 的应用在 Android 和 iOS 的一种跨平台技术，它的特点是共享 iOS 和 Android 应用的逻辑而保持系统原生的用户体验，结合了跨平台和原生开发方式的优点。
	- KMM 是 Kotlin 语言实现的移动端跨平台开发模式，它派生于 Kotlin 生态早已经推出的跨平台解决方案 KMP（Kotlin Multiplatform），它的主要思想是使用不同的编译器编译同一份代码生成各端的不同产物来达到跨平台的目的。KMP 是在 Kotlin 1.2版本提出的，可以将 Kotlin 代码运行到特定平台的 JVM 和 JS 引擎上，分别叫 Kotlin/JVM 和 Kotlin/JS；另外还有将 Kotlin 编译为原生的二进制文件在没有虚拟机的情况下运行的技术 Kotlin/Native，在 Kotlin/Native 中 Kotlin 可以在 iOS 平台直接与 C 以及 Objective-C 代码互操作。KMM 就是利用了 Kotlin/JVM 和 Kotlin/Native 的能力实现的针对 Android 和 iOS 平台的 Kotlin 框架。
	- 从 Kotlin 发展的时间线可以知道，Kotlin 语言得到了 Android 官方的大力支持，2017年I/O上确定 Kotlin 是 Android 官方支持的一等开发语言。在去年10月 Android 官方宣布 Jetpack 开始支持 KMM，目前 Collections 和 DataStore 可以通过依赖 -dev01 版本在 KMM 上使用，同时 KMM 进入了 Beta 版本阶段，预计今年将发布稳定版。
	- ![image.png](../assets/image_1684432799100_0.png)
- # 小试KMM，名不虚传
  collapsed:: true
	- 了解了 KMM 的大概来历和作用，下面就简单试试水，看看如何上手 KMM。
	- ### 开发环境准备
	  collapsed:: true
		- 更新 Android Studio 和 Xcode（用于 iOS 代码开发编译，本文暂略）到最新版本
		  JDK
		  从 Android Studio 下载 KMM 插件
		  更新 Kotlin Plugin 插件到最新版本
	- ### 创建KMM工程
	  collapsed:: true
		- 准备好必须的环境后就可以创建跨平台的App项目了，使用 Android Studio 创建 KMM 项目与我们创建普通的 Android 项目类似，可以创建 KMM 的 App 工程，也可以创建 Library 工程，创建的 App 工程中包含双端的壳工程和 shared 模块，Library 工程中就只包含 shared 模块
		  collapsed:: true
			- ![image.png](../assets/image_1684432840652_0.png)
		- 配置完成应用程序的名称、包名、项目位置及最小SDK版本后，点击下一步进入工程中模块名称的配置，并选择iOS的发布类型，目前支持 CocoaPods Dependency Manager 和 Regular framework 两种，前者用于更复杂的项目，需要使用 CocoaPods 依赖管理器来管理依赖库，后者不需要三方工具，直接通过内部的 Gradle 任务和 Xcode 编译配置来集成 KMM 模块。
		  collapsed:: true
			- ![image.png](../assets/image_1684432852927_0.png)
	- ## 代码工程结构
	  collapsed:: true
		- 创建好的 KMM App 工程中包含三个模块：
		  collapsed:: true
			- androidApp：是 Android 平台的应用程序模块，用于实现 Android App 的 UI 和平台相关的能力。
			- iosApp：是 iOS 应用程序模块，它依赖并使用 shared 模块作为 iOS 框架。
			- shared：是一个 Kotlin 模块，包含 Android 和 iOS 应用程序在平台之间共享的通用代码逻辑。
		- ![image.png](../assets/image_1684432879108_0.png)
		- androidApp 和 shared 两个模块都是 Gradle 项目，在根目录下的 settings.gradle.kts 文件中有被 include，而 iosApp 是 Xcode 项目，需要单独编译，shared 工程需要被打包成 framework 提供给 iOS 使用。
		- 在 shared 模块内包含了三个源码集：androidMain、commonMain 和 iosMain，androidMain和 iosMain 分别是 Android 和 iOS 这两个平台的具体实现的源码集，通过 actual 关键字实现在 commonMain 中以 expected 关键字声明的 API。源码集是一个有逻辑关联的代码集合的 Gradle 概念，在 shared 模块根目录下的 build.gradle.kts 文件中可以为每个源码集配置依赖，Kotlin 标准库会被自动加到相应的 sourceSet 中，无需重复引入。
- # 深交KMM，内力深厚
  collapsed:: true
	- 从前面的了解我们对 KMM 的大致认识是：为 Android 和 iOS 应用程序的网路、数据存储、数据分析和其他通用逻辑维护一份共享的代码库，然后分别实现平台特有的功能，UI 部分也采用给自平台的原生代码实现。简而言之 KMM 实现跨平台的理念就是“Shared business, native UI”。
	- ![image.png](../assets/image_1684432903111_0.png)
- # 接下来我们进一步探索 KMM 的 shared 模块是如何工作的。
	- ### 共享代码库编译过程
		- 首选我们看一下 KMM 的编译过程。shared 模块下的 build.gradle.kts 文件中在 plugins 闭包下加载了 multiplatform 插件，并把 shared module 作为 library 输出。在 kotlin 闭包中配置了需要编译的平台。
		- ```
		  plugins {
		      kotlin("multiplatform")
		      id("com.android.library")
		  }
		  
		  kotlin {
		      android()
		      iosX64()
		      iosArm64()
		      iosSimulatorArm64()
		  }
		  ```