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
	- ## 支持的类
		- Activity、Fragment、View、Service、BroadcastReceiver
	-
- 2、确定哪个类使用依赖注入，添加@AndroidEntryPoint注解。Hilt支持的Android 入口类有：
	- 比如在Activity中注入某个类：
		- ```
		  @AndroidEntryPoint
		  public class ExampleActivity extends AppCompatActivity { ... }
		  ```
- ## 2. 基本使用
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