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
	- 了解了 KMM 的大概来历和作用，下面就简单试试水，看看如何上手 KMM。
	- ### 开发环境准备
	  collapsed:: true
		- 更新 Android Studio 和 Xcode（用于 iOS 代码开发编译，本文暂略）到最新版本
		  JDK
		  从 Android Studio 下载 KMM 插件
		  更新 Kotlin Plugin 插件到最新版本
	- ### 创建KMM工程
		-