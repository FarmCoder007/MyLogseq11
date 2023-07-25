- # 一、编译器常量
  collapsed:: true
	- ```java
	  // java 里 编译器常量  编译时就初始化   后不可修改    static final修饰
	  public static final String path = "path";
	   
	  // kotlin 里 在伴生对象 达到static目的  且 使用const关键字让静态属性  变成 编译器常量  
	  class BaseApplication : Application() {
	      /**
	       *  伴生对象    在该class内部维护一个内部单例类的对象
	       */
	      companion object {
	          // 全局上下文
	          private lateinit var currentContext: Context
	          // 声明编译器常量    const val
	          const val path: String = "path"
	   
	          /**
	           *  定义获取context 的函数
	           */
	          fun currentApplicationContext(): Context {
	              return currentContext
	          }
	      }
	   
	      override fun onCreate() {
	          super.onCreate()
	          currentContext = this
	      }
	  }
	  ```
- # 二、抽象属性
  collapsed:: true
	- 背景：
	  collapsed:: true
		- ```java
		  /**
		   *  author : xu
		   *  date : 2021/3/8 14:35
		   *  description : 声明每个页面的Presenter类型  定义获取 presenter 方法
		   */
		  interface BaseView<T> {
		      fun getPresenter(): T
		  }
		  ```
		- ```java
		  class MainActivityThree : AppCompatActivity(), BaseView<MainThreePresenter> {
		      val mainThreePresenter = MainThreePresenter()
		   
		      /**
		       *  不能每次调用都new一个对象  所以 声明变量 返回
		       */
		      override fun getPresenter(): MainThreePresenter {
		          return mainThreePresenter
		      }
		  }
		  ```
	- 以上声明 可以 通过2种方式 获取 mainThreePresenter     getPresenter()  显得很重复    在Kotlin可以声明抽象属性
	- 在 Kotlin 中，我们可以声明抽象属性，⼦类对抽象属性重写的时候需要重写对应的
	  setter/getter
		- ```java
		  interface BaseView<T> {
		      // 抽象属性 继承T
		      val presenter:T
		  }
		   
		  class MainActivityThree : AppCompatActivity(), BaseView<MainThreePresenter> {
		      val mainThreePresenter = MainThreePresenter()
		      override val presenter: MainThreePresenter
		          get() = mainThreePresenter
		  }
		  ```
- # 三、kotlin  之 委托 by
  collapsed:: true
	- 我们上边例子  只想在 创建一次 在 get里每次都获取那个对象     并不写成上边那么啰嗦
	- ## **属性委托**
		- 有些常⻅的属性操作，我们可以通过委托的⽅式，让它只实现⼀次，例如：
		- ## [[#red]]==**lazy 延迟属性**==：值只在第⼀次访问的时候计算
		  collapsed:: true
			- 1-1通过 by lazy  延迟属性       通过 by将 统一的操作 委托给统一的对象
				- ```java
				  class MainActivityThree : AppCompatActivity(), BaseView<MainThreePresenter> {
				      /**
				       *  by lazy  只有在 第一次访问的时候 才会创建
				       *  调用直接使用presenter
				       */
				      override val presenter: MainThreePresenter by lazy {
				          MainThreePresenter()
				      }
				  }
				  ```
		- observable 可观察属性：属性发⽣改变时的通知
		- map 集合：将属性存在⼀个 map 中
	- ## 注意
		- 对于⼀个只读属性 ( 即 val 声明的 ) ，委托对象必须提供⼀个名为 getValue() 的函数
		- 对于⼀个可变属性 ( 即 var 声明的 ) ，委托对象同时提供 setValue() 、 getValue() 函数
	- # 示例：*多个属性都需要 同样存贮的处理*
		- 背景
		  collapsed:: true
			- ```java
			    // 多个属性都需要 同样存贮的处理  
			    var token:String
			          get() {
			              return Sp.getValue("token")
			          }
			          set(value) {
			              // 每次改变的时候  存本地
			              Sp.save("token",value);
			          }
			      var token2:String
			          get() {
			              return Sp.getValue("token2")
			          }
			          set(value) {
			              // 每次改变的时候  存本地
			              Sp.save("token2",value);
			          }
			  ```
		- 委托
			- ```java
			      /**
			       *  属性委托 通过 by关键字   后边需要接对象
			       */
			      var token:String by Saver("token")
			      var token2:String by Saver("token2")
			      /**
			       *  这个委托的对象  因为属性的变量类型是var  需要 有两个方法getValue setValue
			       *                                val   则需要有一个 getValue的方法即可
			       *  可以通过构造 把Key传入
			       */
			      class Saver(var token:String){
			          operator fun getValue(mainActivityThree: MainActivityThree, property: KProperty<*>): String {
			              //  接收的不可空类型   在变量后加！！为非空判断
			              return Sp.getValue(token)!!
			          }
			   
			          operator fun setValue(mainActivityThree: MainActivityThree, property: KProperty<*>, s: String) {
			              Sp.save(token,value);
			          }
			      }
			  ```
- # 三、变量，常量 声明  var 和 val关键字
	- ## 变量：
	  collapsed:: true
		- ```java
		  //java:
		  int age = 19;
		  // kotlin  变量var关键字    age变量名    ：  变量类型
		  var age: Int = 19
		   
		   
		   
		  // java
		  final String  name = "java";
		  // kotlin 常量 val相当于 final 关键字      首次初始化变量 可以省略变量类型 kotlin自动识别
		  val name = "kotlin";
		  
		  // java 
		  private final String passwordKey = "password";
		  // kotlin 有类型推断  变量类型String可省略
		  private val passwordKey = "password"
		  ```
	- ## 对象声明
	  collapsed:: true
		- ```java
		  // java  new关键字
		  Test test = new Test();
		  // kotlin 不需要new   直接 调用构造函数
		  var test : Test = Test()
		  ```