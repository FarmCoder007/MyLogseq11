# [官网](https://dagger.dev/hilt/gradle-setup)
- # [Android开发者教程](https://developer.android.com/training/dependency-injection/hilt-android?hl=zh-cn)
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
- ## 不支持的
	-
- [[Hilt]]
- [[浅析Jetpack成员-Android依赖注入框架Hilt]]
-