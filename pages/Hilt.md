- ## hilt的依赖注入
  collapsed:: true
	- 铺垫了这么依赖注入，终于来到了Hilt的介绍。
	- Hilt 是 Google 专门针对 Android 平台做的一个依赖注入库。它不是从里到外全新开发的，而是基于 Dagger 做的，它的下层还是 Dagger。
	- Hilt意为刀柄，而Dagger是匕首的意思。匕首很好用，但使用门槛高，刀柄拿住就好。
	- 为什么不直接去优化改进 Dagger，而要基于它做一个新库呢？因为 Hilt 做的事其实也并不是对 Dagger 进行优化，而是场景化：针对 Android 开发制定了一系列的规则，通过这些规则大大简化了这套工具的使用。比如Hilt 会自动找到 Android 的系统组件里面那些最佳的初始化位置——比如 Activity 的 onCreate() ——然后在这些位置注入依赖。而Dagger使用很繁琐且复杂。
	- 现在回到上文中的问题，如何依赖注入？
		- ## 1. 添加依赖
			- 在项目的根目录的build.gradle添加
			  collapsed:: true
				- ```
				  buildscript {
				      ...
				      dependencies {
				          ...
				          classpath 'com.google.dagger:hilt-android-gradle-plugin:2.28-alpha'
				      }
				  }
				  ```
			- 然后在app/build.gradle文件中添加以下依赖项：
			  collapsed:: true
				- ```
				  ...
				  apply plugin: 'kotlin-kapt'
				  apply plugin: 'dagger.hilt.android.plugin'
				  
				  android {
				      ...
				  }
				  
				  dependencies {
				      implementation "com.google.dagger:hilt-android:2.28-alpha"
				      kapt "com.google.dagger:hilt-android-compiler:2.28-alpha"
				  }
				  ```
	-