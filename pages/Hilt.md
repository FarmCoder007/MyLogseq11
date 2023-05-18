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
			- Hilt支持的依赖注入注解有： @HiltAndroidApp、 @AndroidEntryPoint 、@Module、@InstallIn 、@Provides以及Java Inject包中的几个注解：Inject、Qualifier、Scope、Singleton
		- ## 2. 使用步骤
		  collapsed:: true
			- 1、在工程Application添加@HiltAndroidApp注解
			  collapsed:: true
				- ```
				  @HiltAndroidApp
				  public class ExampleApplication extends Application { ... }
				  ```
			- 2、确定哪个类使用依赖注入，添加@AndroidEntryPoint注解。Hilt支持的Android 入口类有：Activity、Fragment、View、Service、BroadcastReceiver
			  collapsed:: true
				- 比如在Activity中注入某个类：
					- ```
					  @AndroidEntryPoint
					  public class ExampleActivity extends AppCompatActivity { ... }
					  ```
			- 3、注入类
			  collapsed:: true
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
				  ```
			-
		-
		-
	-