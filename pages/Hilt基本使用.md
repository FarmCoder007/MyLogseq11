# 1、Application设置（必须）
collapsed:: true
	- 所有使用 Hilt 的应用都必须包含一个带有 `@HiltAndroidApp` 注解的 [`Application`](https://developer.android.com/reference/android/app/Application?hl=zh-cn) 类。
	- `@HiltAndroidApp` 会触发 Hilt 的代码生成操作，生成的代码包括应用的一个基类，该基类充当应用级依赖项容器
	- 在工程Application添加@HiltAndroidApp注解。
		- ```kotlin
		  @HiltAndroidApp
		  class ExampleApplication : Application() { ... }
		  ```
- # 2、依赖项注入Android类
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
- # 3、注意
  collapsed:: true
	- 由 Hilt 注入的字段不能为私有字段。正确示例
		- ```kotlin
		  @AndroidEntryPoint
		  class ExampleActivity : AppCompatActivity() {
		  
		    @Inject lateinit var analytics: AnalyticsAdapter
		    ...
		  }
		  ```
- # 4、除构造函数注入方式外-Hilt模块
	- ## 无法使用构造函数注入的场景
		- 1、接口没有构造函数，不能通过构造函数注入接口
		- 2、不能使用构造函数注入三方库的类，不属于你的类
	- ## 使用 Hilt 模块向 Hilt 提供绑定信息
		- 1、 `@Module` 注解标记模块类，它会告知 Hilt 如何提供某些类型的实例
		- 2、使用 `@InstallIn` 为 Hilt 模块添加注解，以告知 Hilt 每个模块将用在或安装在哪个 Android 类中
	- ##  4-1、使用 @Binds 注入接口实例
	  collapsed:: true
		- 需求：
			- 将AnalyticsService接口注入到ExampleActivity类中
		- 实现：
			- 无法通过注入构造函数方式，接口没有构造函数，
			- 1、@Module标记AnalyticsModule，作用：通过此模块提供注入方式
			- 2、@InstallIn(ActivityComponent::class) 作用：该模块应用到 ExampleActivity 上对应组件就是ActivityComponent
			- 3、Hilt 模块内创建一个带有 `@Binds` 注解的抽象函数 - 来注入接口实例    `@Binds` 注解会告知 Hilt 在需要提供接口的实例时要使用哪种实现
				- 带有注解的函数会向 Hilt 提供以下信息：
				- 函数返回类型会告知 Hilt 该函数提供哪个接口的实例。
				- 函数参数会告知 Hilt 要提供哪种实现
		- 代码示例
			- ```kotlin
			  // 将该接口注入到
			  interface AnalyticsService {
			    fun analyticsMethods()
			  }
			  
			  // 构造函数被注入，因为Hilt需要知道如何 提供AnalyticsServiceImpl的实例
			  class AnalyticsServiceImpl @Inject constructor(
			    ...
			  ) : AnalyticsService { ... }
			  
			  @Module
			  @InstallIn(ActivityComponent::class)
			  abstract class AnalyticsModule {
			  
			    @Binds
			    abstract fun bindAnalyticsService(
			      analyticsServiceImpl: AnalyticsServiceImpl
			    ): AnalyticsService
			  }
			  ```
	- ## 4-2、使用 @Provides 注入外部三方库实例
		- 如果 `AnalyticsService` 类不直接归您所有，您可以告知 Hilt 如何提供此类型的实例，方法是在 Hilt 模块内创建一个函数，并使用 `@Provides` 为该函数添加注解
			- - 函数返回类型会告知 Hilt 函数提供哪个类型的实例。``
			- - 函数参数会告知 Hilt 相应类型的依赖项。
			- - 函数主体会告知 Hilt 如何提供相应类型的实例。每当需要提供该类型的实例时，Hilt 都会执行函数主体。
		- ### 1、单一创建实例的方式
		  collapsed:: true
			- ```kotlin
			  @Module
			  @InstallIn(ActivityComponent::class)
			  object AnalyticsModule {
			  
			    // 1、声明注入的类型：AnalyticsService
			    // 2、函数体 具体创建实例的方式
			    // 3、函数参数：可传入 构建实例时需要的依赖项/参数
			    @Provides
			    fun provideAnalyticsService(
			      // 此类型的潜在依赖项，通过入参传入
			    ): AnalyticsService {
			        return Retrofit.Builder()
			                 .baseUrl("https://example.com")
			                 .build()
			                 .create(AnalyticsService::class.java)
			    }
			  }
			  ```
		- ### 2、自定义注解标记[[#red]]==**多个创建实例的方式**==
			- 背景
				- 需要让 Hilt 以依赖项的形式提供同一类型的不同实现，必须向 Hilt 提供多个绑定
			- ### 步骤1、@Qualifier声明`不同实现方式`的注解
				- ```kotlin
				  @Qualifier
				  @Retention(AnnotationRetention.BINARY)
				  annotation class AuthInterceptorOkHttpClient
				  
				  @Qualifier
				  @Retention(AnnotationRetention.BINARY)
				  annotation class OtherInterceptorOkHttpClient
				  ```
			- ### 步骤2、带有 `@Provides` 的 Hilt 模块中声明不同的返回类型
			- ### 步骤3、注入类中也通过自定义注解，使用对应类型