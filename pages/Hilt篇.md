# [官网](https://dagger.dev/hilt/gradle-setup)
- # [Android开发者教程](https://developer.android.com/training/dependency-injection/hilt-android?hl=zh-cn)
- # 简介
  collapsed:: true
	- 铺垫了这么依赖注入，终于来到了Hilt的介绍。
	- Hilt 是 Google 专门针对 Android 平台做的一个依赖注入库。它不是从里到外全新开发的，而是基于 Dagger 做的，它的下层还是 Dagger。
	- Hilt意为刀柄，而Dagger是匕首的意思。匕首很好用，但使用门槛高，刀柄拿住就好。
	- 为什么不直接去优化改进 Dagger，而要基于它做一个新库呢？因为 Hilt 做的事其实也并不是对 Dagger 进行优化，而是场景化：针对 Android 开发制定了一系列的规则，通过这些规则大大简化了这套工具的使用。比如Hilt 会自动找到 Android 的系统组件里面那些最佳的初始化位置——比如 Activity 的 onCreate() ——然后在这些位置注入依赖。而Dagger使用很繁琐且复杂。
- # 一、[[Hilt基本使用]]
- # 二、Hilt注入项创建时机
	- ## 1、@AndroidEntryPoint标记的组件，内部通过`@Inject` 的标记依赖项，依赖项通过`@Inject` 标记的构造函数时
	  collapsed:: true
		- 当一个组件（例如 `Activity`）上使用 `@AndroidEntryPoint` 注解，并且在该组件中存在通过 `@Inject` 标记的构造函数时，这些被标记为 `@Inject` 的依赖项（比如 `Test` 类）会在该Activity被创建的时候被 Dagger Hilt 框架自动创建和注入
		- ```kotlin
		  @AndroidEntryPoint
		  class MyActivity : AppCompatActivity() {
		  
		      @Inject
		      lateinit var test: Test
		  
		      // ...
		  
		      override fun onCreate(savedInstanceState: Bundle?) {
		          super.onCreate(savedInstanceState)
		          setContentView(R.layout.activity_main)
		  
		          // 此时 test 对象已经被 Dagger Hilt 创建和注入了
		          // 你可以在这里使用 test 对象
		      }
		  }
		  
		  ```
		- [[#red]]==**Test 类的实例会在 MyActivity 被创建时由 Dagger Hilt 框架自动创建，并且会被注入到 MyActivity 中的 test 字段中**==。这确保了在 `MyActivity` 使用 `test` 对象时，该对象已经被正确地初始化和注入。
- [[Hilt]]
- [[浅析Jetpack成员-Android依赖注入框架Hilt]]
-