# [官网](https://dagger.dev/hilt/gradle-setup)
- # [Android开发者教程](https://developer.android.com/training/dependency-injection/hilt-android?hl=zh-cn)
- # 一、[[Hilt基本使用]]
- # 二、Hilt注入项创建时机
	- ## 1、@AndroidEntryPoint标记的组件，内部通过`@Inject` 的标记依赖项，依赖项通过`@Inject` 标记的构造函数时
		- 当一个组件（例如 `Activity`）上使用 `@AndroidEntryPoint` 注解，并且在该组件中存在通过 `@Inject` 标记的构造函数时，这些被标记为 `@Inject` 的依赖项（比如 `Test` 类）会在该被创建的时候被 Dagger Hilt 框架自动创建和注入
		- ```kotlin
		  ```
- ## 不支持的
	-
- [[Hilt]]
- [[浅析Jetpack成员-Android依赖注入框架Hilt]]
-