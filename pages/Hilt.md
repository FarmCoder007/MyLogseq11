## hilt的依赖注入
	- 铺垫了这么依赖注入，终于来到了Hilt的介绍。
	- Hilt 是 Google 专门针对 Android 平台做的一个依赖注入库。它不是从里到外全新开发的，而是基于 Dagger 做的，它的下层还是 Dagger。
	- Hilt意为刀柄，而Dagger是匕首的意思。匕首很好用，但使用门槛高，刀柄拿住就好。
	- 为什么不直接去优化改进 Dagger，而要基于它做一个新库呢？因为 Hilt 做的事其实也并不是对 Dagger 进行优化，而是场景化：针对 Android 开发制定了一系列的规则，通过这些规则大大简化了这套工具的使用。比如Hilt 会自动找到 Android 的系统组件里面那些最佳的初始化位置——比如 Activity 的 onCreate() ——然后在这些位置注入依赖。而Dagger使用很繁琐且复杂。
	- 现在回到上文中的问题，如何依赖注入？
		- ## 1. 添加依赖
		  collapsed:: true
			- 在项目的根目录的build.gradle添加
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
			- Hilt支持的依赖注入注解有： @HiltAndroidApp、 @AndroidEntryPoint 、@Module、@InstallIn 、@Provides以及Java Inject包中的几个注解：Inject、Qualifier、Scope、Singleton
		- ## 2. 使用步骤
			- 1、在工程Application添加@HiltAndroidApp注解
				- ```
				  @HiltAndroidApp
				  public class ExampleApplication extends Application { ... }
				  ```
			- 2、确定哪个类使用依赖注入，添加@AndroidEntryPoint注解。Hilt支持的Android 入口类有：Activity、Fragment、View、Service、BroadcastReceiver
				- 比如在Activity中注入某个类：
					- ```
					  @AndroidEntryPoint
					  public class ExampleActivity extends AppCompatActivity { ... }
					  ```
			- 3、注入类
				- 在组件中获取依赖项，需要使用@Inject注解标记字段注入，以上面的例子为例，在Activity中注入一个User对象，应该如下：
				- ```
				  @AndroidEntryPoint
				  public class ExampleActivity extends AppCompatActivity {
				  
				    @Inject
				    User currentUser;
				    ...
				  }
				  ```
			- 4、定义注入类注入方式
			  collapsed:: true
				- 为了执行字段注入，Hilt需要知道如何从组件中获得注入对象。这里以最简单的构造函数注入方式为例，需要在类的构造函数中添加@Inject注解：
					- ```
					  class User{
					    private String name;
					    private int age;
					    
					    @Inject
					    public User(){
					      this.name = "Tom";
					      this.age = 18;
					    }
					  }
					  ```
			- 按照以上步骤就可以实现Hilt简单的依赖注入了。我们在实际的使用中也不仅仅是这么简单的情况，下面介绍一下其他情况如何进行注入。
		- ## 3. 进阶使用
			- ### Hilt模块
				- 对于简单无参的构造函数类，可以直接使用@Inject注解进行注入对象，但对于有参数的类、接口或我们无法修改的第三方类，就需要自定义一个带有@Module注解的类来告知Hilt如何在外部进行初始化该类对象。同时需要对该模块类添加@InstallIn注解，告知Hilt这个模块类应用在哪个Android类中。
				- ```
				  @Module
				  @InstallIn(ApplicationComponent::class)
				  class NetworkModule {
				  
				      //...省略...
				  
				  }
				  ```
			- ### 带参数的对象注入
			  collapsed:: true
				- 一般类总会有带参数的构造函数，或者初始化类对象后需要再次设置一些成员变量。这时候我们就需要使用**@Provides**注解来告知Hilt如何构造当前类对象。
				- ```
				  //注入类，构造函数配合provides注解的话，不用添加@Inject注解
				  class OldDriver constructor(val name:String) {
				  
				  }
				  -----
				  
				  @Module
				  @InstallIn(ActivityComponent::class)
				  class ProvideModule {
				  
				    	//使用Provides注解来初始化OldDriver类
				      @Provides
				      fun getDriver(): OldDriver {
				          return OldDriver("Tony")
				      }
				  }
				  ```
			- ### 接口注入
			  collapsed:: true
				- 接口注入和普通类注入不太相同，接口注入需要将Hilt模块类声明为抽象类，使用**@Binds**注解修饰获取对象的方法。以一个简单的View#OnClickListener为例，首先需要定义一个类实现OnClickListener接口，在默认构造函数中添加@Inject接口
				- ```
				  class CustomOnClickListener @Inject constructor() : View.OnClickListener {
				      override fun onClick(v: View?) {
				          Toast.makeText(v?.context, "click this view", Toast.LENGTH_SHORT).show()
				      }
				  }
				  ```
				- 创建一个抽象类模块，添加@Module和@InstallIn注释，将CustomOnClickListener设置到方法参数中，返回值为接口类，使用@Binds注解修饰
				- ```
				  @Module
				  @InstallIn(ActivityComponent::class)
				  abstract class AbsModule {
				  
				      @Binds
				      abstract fun getOnClickListener(listener: CustomOnClickListener):View.OnClickListener
				  }
				  ```
			- ### 第三方类的依赖注入
			  collapsed:: true
				- 一般我们在项目中，自定义的类可以直接使用@Inject注解直接修饰，如果是外部类无法修改他的源码添加注解，这时候做法和上面的注入带参数构造函数方法一致，也是使用**@Provides注解配合@Module**注解一起使用。以一个注入Retrofit为例：
				- ```
				  @Module
				  @InstallIn(ApplicationComponent::class)
				  class NetworkModule {
				  
				      @Singleton
				      @Provides
				      fun provideOkHttpClient(): OkHttpClient {
				          return OkHttpClient.Builder()
				              .connectTimeout(20, TimeUnit.SECONDS)
				              .readTimeout(20, TimeUnit.SECONDS)
				              .writeTimeout(20, TimeUnit.SECONDS)
				              .build()
				      }
				  
				      @Singleton
				      @Provides
				      fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
				          return Retrofit.Builder()
				              .addConverterFactory(GsonConverterFactory.create())
				              .baseUrl("https://app.website.com/")
				              .client(okHttpClient)
				              .build()
				      }
				  }
				  ```
				-
			- ### 注入相同类的不同实例
			  collapsed:: true
				- 如果现在需要获取同一个接口的不同实例。上面讲到了使用 @Binds 注入接口实例，但是现在需要多个不同的实例，不能通过设置不同的方法名来注入多个实例。
				- 1、首先要使用**@Qualifier** 和**@Retention**注解来自定义注释的限定符，这个限定符用于模块的@Binds、@Provides注解
				  collapsed:: true
					- ```
					  @Qualifier
					  @Retention(RetentionPolicy.RUNTIME)
					  private annotation class AuthInterceptorOkHttpClient
					  
					  @Qualifier
					  @Retention(RetentionPolicy.RUNTIME)
					  private annotation class OtherInterceptorOkHttpClient
					  ```
				- 2、然后，Hilt需要知道如何提供与每个限定符对应的类型的实例。下面的两个方法具有相同的返回值类型，但是限定符将它们标记为两个不同的绑定。
				  collapsed:: true
					- ```
					  @Module
					  @InstallIn(ApplicationComponent::class)
					  class NetworkModule {
					      @AuthInterceptorOkHttpClient
					      @Provides
					      fun provideAuthInterceptorOkHttpClient(AuthInterceptor authInterceptor): OkHttpClient {
					          return OkHttpClient.Builder()
					              .addInterceptor(authInterceptor)
					              .build();
					      }
					  
					      @OtherInterceptorOkHttpClient
					      @Provides
					      fun provideOtherInterceptorOkHttpClient(OtherInterceptor otherInterceptor): OkHttpClient {
					          return OkHttpClient.Builder()
					              .addInterceptor(otherInterceptor)
					              .build();
					      }
					  }
					  ```
				- 3、在使用时也需要使用自定义的限定符来标记不同的实例
				  collapsed:: true
					- ```
					  @AndroidEntryPoint
					  class ExampleActivity : AppCompatActivity() {
					  
					    @AuthInterceptorOkHttpClient
					    @Inject
					    val okHttpClient:OkHttpClient
					    ...
					  }
					  ```
			- ### 注入类的生命周期和作用域
				- 组件的生命周期限制了组件创建和销毁之间的作用域，其次生命周期指示何时可以使用成员注入的值。
				- 组件生命周期通常受限于 Android 类的相应实例的创建和销毁。下表列出了每个组件的范围注释和有界生命周期。
				- |  == Component ==   | ==注入对象==  | ==作用域==  | ==创建于==  |
				  |  ApplicationComponent  | Application  |@Singleton|Application#onCreate()|
				  |  ActivityRetainedComponent  | ViewModel  |@ActivityRetainedScoped|Activity#onCreate()|
				  |  ViewModelComponent  | ViewModel  |@ViewModelScoped|ViewModel created| 
				  |  ActivityComponent  | Activity  |@ActivityScoped|Activity#onCreate()|
				  |  FragmentComponent  | Fragment  |@FragmentScoped|Fragment#onAttach()|
				  |  ViewComponent  | View  |@ViewScoped|View#super()|
				  |  ViewWithFragmentComponent  | View with @WithFragmentBindings  |@ViewScoped|View#super()|
				  |  ServiceComponent  | Service  |@ServiceScoped|Service#onCreate()|
				- 一般情况下，我们在设置注入对象时仅使用**@Inject**来标记，此时注入对象是无作用域的，这意味着每次都会注入一个新的对象。如果我们在注入方法的上面添加Scope相关的注解，作用域绑定只会在他作用域范围内的组件中生成一个实例，举个例子来说：
					- ```
					  // 仅有Inject注解注入是无作用域的，每一次请求都会创建一个新的实例
					  class UnscopedBinding @Inject constructor() {
					  }
					  
					  //有ActivityScoped注解修饰的对象，在同一个Activity中的实例是唯一的。
					  @ActivityScoped
					  class ScopedBinding @Inject constructor() {
					  }
					  ```
				- 有ActivityScoped注解修饰的对象，在同一个Activity中的实例是唯一的，不同的Activity还是会创建新的对象。从上面的表格中可以看到Hilt对于注入对象的作用域支持很多：有常见的全局唯一单例类型的对象，也有和ViewModel声明周期对应的作用域。不同的作用域这样就可解决多对象单例过多占用资源的问题。
		-
		-
	-
- 总结
	- 文章介绍了依赖注入的概念和Hilt库的使用。依赖注入就是：由外部初始化依赖的类对象，而不是类内部去创建对象。我们日常使用的构造函数初始化对象，Builder模式，工厂模式都是依赖注入的方式。Hilt库只是帮助我们简化设置对象的过程，并提供对象作用域的能力。
	- Hilt库必须要在Application中添加@HiltAndroidApp然后在注入类和对象类中使用@Inject注解来简单实现对象注入，也枚举了一些诸如接口注入、第三方类的注入等情况。
- 题外话
  Hilt优点说了挺多比如：减少模板代码、分离依赖项、重构容易等等，Google也将这个Hilt库添加到Jetpack中来推广并支持依赖注入。
- 那为什么我们不用依赖注入库？
- 我认为的原因有这些：
- 对于依赖注入名称不理解
  对于依赖注入库有一定的学习成本
  依赖注入会增加打包时间，降低性能
  对于Hilt性能，网上有作者从项目结构、代码行数、编译时间等角度将Hilt和纯 Kotlin 的注入库Koin做了对比。（文章在这里）结论是由于Hilt使用了注解和kapt的代码编译时生成，会导致使用了Hilt库代码行数远多于没有额外代码生成的Koin，编译时间也会大大增加，因此在大型项目中Hilt引用也会很谨慎。
- 参考资料
- https://developer.android.com/training/dependency-injection/hilt-android?hl=zh_cn
- https://dagger.dev/hilt/
- https://rengwuxian.com/cong-dagger-dao-hilt-gu-ge-wei-shi-yao-zhi-zhao-yu-rang-wo-men-shi-yong-yi-lai-zhu-ru/
- https://juejin.cn/post/6986450257295081502