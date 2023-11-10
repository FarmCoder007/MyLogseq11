# [官网](https://dagger.dev/hilt/gradle-setup)
- # [Android开发者教程](https://developer.android.com/training/dependency-injection/hilt-android?hl=zh-cn)
- # 使用
	- ## 1、Application设置（必须）
	  collapsed:: true
		- 所有使用 Hilt 的应用都必须包含一个带有 `@HiltAndroidApp` 注解的 [`Application`](https://developer.android.com/reference/android/app/Application?hl=zh-cn) 类。
		- `@HiltAndroidApp` 会触发 Hilt 的代码生成操作，生成的代码包括应用的一个基类，该基类充当应用级依赖项容器
		- 在工程Application添加@HiltAndroidApp注解。
			- ```kotlin
			  @HiltAndroidApp
			  class ExampleApplication : Application() { ... }
			  ```
	- ## 2】
- [[Hilt]]
- [[浅析Jetpack成员-Android依赖注入框架Hilt]]
-