## 1、Application设置（必须）
collapsed:: true
	- 所有使用 Hilt 的应用都必须包含一个带有 `@HiltAndroidApp` 注解的 [`Application`](https://developer.android.com/reference/android/app/Application?hl=zh-cn) 类。
	- `@HiltAndroidApp` 会触发 Hilt 的代码生成操作，生成的代码包括应用的一个基类，该基类充当应用级依赖项容器
	- 在工程Application添加@HiltAndroidApp注解。
		- ```kotlin
		  @HiltAndroidApp
		  class ExampleApplication : Application() { ... }
		  ```
- ## 2、依赖项注入Android类
  collapsed:: true
	- ## 支持的类
	  collapsed:: true
		- Application（通过使用 `@HiltAndroidApp`）
		- ViewModel（通过使用 `@HiltViewModel`）
		- Activity、Fragment、View、Service、BroadcastReceiver (通过使用@AndroidEntryPoint)
	- ## 不支持的类
	  collapsed:: true
		- - Hilt 仅支持扩展 [`ComponentActivity`](https://developer.android.com/reference/kotlin/androidx/activity/ComponentActivity?hl=zh-cn) 的 activity，如 [`AppCompatActivity`](https://developer.android.com/reference/kotlin/androidx/appcompat/app/AppCompatActivity?hl=zh-cn)。
		- - Hilt 仅支持扩展 `androidx.Fragment` 的 Fragment。
		- - Hilt 不支持保留的 fragment。
	- ## 使用
	  collapsed:: true
		- 确定哪个类使用依赖注入，添加@AndroidEntryPoint注解
		- 比如在Activity中注入AnalyticsAdapter类：
			- 1、@AndroidEntryPoint 注解标记要注入的类2、@Inject标记字段注入
				- ```kotlin
				  @AndroidEntryPoint
				  class ExampleActivity : AppCompatActivity() {
				  
				    @Inject lateinit var analytics: AnalyticsAdapter
				    ...
				  }
				  ```
			- 3、定义注入类AnalyticsAdapter注入方式
				- 1、简单的构造函数注入
				  collapsed:: true
					- 为了执行字段注入，Hilt需要知道如何从组件中获得注入对象。这里以最简单的构造函数注入方式为例，需要在类的==**构造函数中添加@Inject注解**==：
					- ```kotlin
					  class AnalyticsAdapter @Inject constructor(
					    private val service: AnalyticsService
					  ) { ... }
					  ```
- ## 3、注意
  collapsed:: true
	- 由 Hilt 注入的字段不能为私有字段。正确示例
		- ```kotlin
		  @AndroidEntryPoint
		  class ExampleActivity : AppCompatActivity() {
		  
		    @Inject lateinit var analytics: AnalyticsAdapter
		    ...
		  }
		  ```
- ## 4、除构造函数注入方式外-其他注入方式
	- 无法使用构造函数注入的场景
		- 1、接口没有构造函数，不能通过构造函数注入接口
		- 2、不能使用构造函数注入三方库的类
	- ## 1、使用 Hilt 模块向 Hilt 提供绑定信息
		- Hilt 模块是一个带有 `@Module` 注解的类
		- 使用 `@InstallIn` 为 Hilt 模块添加注解，以告知 Hilt 每个模块将用在或安装在哪个 Android 类中