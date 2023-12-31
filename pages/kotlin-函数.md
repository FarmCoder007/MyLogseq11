## 一、函数声明  fun关键字
collapsed:: true
	- ### 无返回值 无 参数函数
	  collapsed:: true
		- ```java
		  // java    
		  public void method(){
		      println("hello word")   
		  }
		  // kotlin
		  /**
		   *  函数的定义  无参数 无返回值类型在java 里用void 声明  在kotlin 用Unit（可以简化不写）
		   *              关键字: fun
		   *             函数名：main
		   *
		   */
		  fun main(): Unit {
		      println("hello word")
		  }
		  ```
	- ### 有参数有返回值的
	  collapsed:: true
		- ```java
		  // java
		  public int doubleNumber(int x) {
		      return 2 * x;
		  }
		   
		  /**
		   *  函数定义  有参数   有返回值
		   *           关键字： fun
		   *           函数名： doubleNumber
		   *           （）里定义参数   （参数名：参数类型）
		   *           返回值类型在函数定义的右边 ：Int
		   */
		  fun doubleNumber(x: Int): Int {
		      return x * 2;
		  }
		  ```
- ## 一、[[kotlin方法中添加默认参数]]
- ## 二、[[kotlin  扩展函数]]
- ## 三、[[内联函数:inline关键字]]
  collapsed:: true
	-
- ## 五、内联函数+reified     实现具体化的类型参数   【设计base用到】
  collapsed:: true
	- 示例
	  collapsed:: true
		- ```java
		  /**
		   *  retrofit 定义 具体请求网络的接口
		   */
		  interface API {
		      @GET("User")
		      fun getUserInfo(): Call<Any>
		  }
		   
		  val RETROFIT = Retrofit.Builder()
		          .baseUrl("https://api.test.com/")
		          .build()
		   
		  /**
		   *  声明泛型T
		   *  clazz ： 传入 某个类的class
		   *  返回 一个代理对象  T
		   */
		  fun <T> create(clazz: Class<T>): T {
		      return RETROFIT.create(clazz)
		  }
		   
		  fun main() {
		      /**
		       *  传入接口的class对象  给retrofit
		       *  使用的时候  总是得把 ::class.java对象传过去  但是有用的只有API
		       *  而 又不能直接把 API接口 这个类型传过去  不能使用泛型获取class  不能T.class
		       */
		      val api = create(API::class.java)
		  }
		  ```
	- inline + reified         由于 inline会把函数体的代码 插入到每个调用处     从而 可以知道 调用方的类型
		- ```java
		  /**
		   *  retrofit 定义 具体请求网络的接口
		   */
		  interface API {
		      @GET("User")
		      fun getUserInfo(): Call<Any>
		  }
		   
		  val RETROFIT = Retrofit.Builder()
		          .baseUrl("https://api.test.com/")
		          .build()
		   
		  /**
		   *  inline  声明称内联函数
		   *  reified 修饰泛型  这样 就可以 获取 泛型T的属性了
		   */
		  inline fun <reified T> create(): T {
		      return RETROFIT.create(T::class.java)
		  }
		   
		  fun main() {
		      // 只需要声明 泛型类型即可
		      val api = create<API>()
		  }
		  ```
- ## 四、java中是不能将函数作为参数传递的    kotlin是可以声明函数 参数   也是可以传递函数的
  collapsed:: true
	- 例如
	  collapsed:: true
		- ```java
		  class View{
		      // 定义点击接口
		      interface OnClickListener{
		          fun onClick(view: View)
		      }
		   
		      // 设置回调
		      fun setOnClickListener(listener: OnClickListener){
		   
		      }
		   
		  }
		   
		  fun main(){
		      val view = View()
		      view.setOnClickListener(object : View.OnClickListener{
		          override fun onClick(view: View) {
		              println("点击了")
		          }
		      })
		  }
		  ```
	- 传入 函数 可以定义**传递函数的引用**   通过：：        定义接收函数
	  collapsed:: true
		- ```java
		   
		  class View {
		  //    // 定义点击接口
		  //    interface OnClickListener{
		  //        fun onClick(view: View)
		  //    }
		      // 设置回调  参数是 函数类型
		      // 1 listener变量名    函数类型的声明是  （函数的参数接收类型 View） ->  函数返回值类型
		      // 2 定义了函数类型  取缔了上边的接口
		      fun setOnClickListener(listener: (View) -> Unit) {
		   
		      }
		   
		  }
		   
		  //定义要传的函数
		  fun onClick(view: View) {
		      println("点击了")
		  }
		   
		  fun main() {
		      val view = View()
		      // :: 代表输入函数的引用
		      view.setOnClickListener(::onClick)
		  }
		  ```
	- 接收类型是函数类型  最简单是lambda  形式 接收函数
	  collapsed:: true
		- ```java
		  fun main() {
		      val view = View()
		   
		  //    view.setOnClickListener({
		  //        println("点击了")
		  //    })
		  //    view.setOnClickListener(){
		  //        println("点击了")
		  //    }
		      view.setOnClickListener{
		          println("点击了")
		      }
		  }
		  ```
	- java中没有 传入 函数类型的     怎么在java调用该方法
		- ```java
		  new View().setOnClickListener(new Function1<View, Unit>() {
		              @Override
		              public Unit invoke(View view) {
		                  return null;
		              }
		          });
		  ```
	- 在 java中 需要传入 Function1 的接口   这是因为 在kotlin 定义声明函数类型的方法时    已经预设了很多 对应的接口  则      接收函数类型的方法  其实接收的是接口
- ## 七、静态函数   kotlin没有static 关键字     三种方法实现 静态函数
  collapsed:: true
	- ## 方法一 ： [[将静态变量 或者 静态函数 写在 kotlin 文件里 而不是类里]]
	- ## 方式二：[[使用object声明kotlin 类]]
	- ## 方式三：伴生对象
	  collapsed:: true
		- 当我们想使用BaseApplication,方法名 调用  application 里的静态方法的时候    我们是不能用方法二 实现的   不能Object直接创建一个单例对象   application 需要framwork层创建  那么使用 伴生对象方法
		- ```java
		  // kotlin  在class内部声明个伴生对象
		  class BaseApplication : Application() {
		      /**
		       *  伴生对象    在该class内部维护一个内部单例类的对象
		       */
		      companion object {
		          // 全局上下文
		          private lateinit var currentContext: Context
		   
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
		   
		   
		  // 调用    通过 Companion
		  BaseApplication.Companion.currentApplicationContext();
		   
		   
		   
		  // 伴生对象 实际调用内部单例对象     如果想真正的生成 static 静态调用      在 Object修饰的类中 使用@JvmStatic修饰函数
		   
		  // kotlin  在class内部声明个伴生对象
		  class BaseApplication : Application() {
		      /**
		       *  伴生对象    在该class内部维护一个内部单例类的对象
		       */
		      companion object {
		          // 全局上下文
		          private lateinit var currentContext: Context
		   
		          /**
		           *  @JvmStatic  只能修饰 object声明的类函数   变量
		           */
		          @JvmStatic
		          fun currentApplicationContext(): Context {
		              return currentContext
		          }
		      }
		   
		      override fun onCreate() {
		          super.onCreate()
		          currentContext = this
		      }
		  }
		   
		  // 调用 直接 类名
		  BaseApplication.currentApplicationContext();
		  ```
- ## 八、扩展函数
  collapsed:: true
	- ```java
	  canvas.save()
	   .. 执行代码
	   
	  canvas.restore()
	  等价于  kotlin的扩展函数
	   
	  canvas.withSave {
	      ...执行代码
	  }
	  ```
- ## 九、kotlin类型推断  简化 函数的声明
  collapsed:: true
	- ```java
	  // 默认函数
	  fun get(key: String?): String? {
	      return SP.getString(key,null)
	  }
	   
	  // return  一行代码就能返回的情况  可以  = 代替return  
	  fun get(key: String?): String? = SP.getString(key,null)
	   
	  // 还可以省略 返回值类型  kotlin 返回函数这样写 可以使用类型推断
	  fun get3(key: String?) = SP.getString(key,null)
	   
	   
	  ```
	- ```kotlin
	  // 没有返回值的也可以直接用 = 连接起来
	  fun save(key:String,value:String){
	      SP.edit().putString(key,value).apply()
	  }
	  // 省略写法
	  fun save(key:String,value:String) =  SP.edit().putString(key,value).apply()
	  ```
- ## 九、[[vararg可变参数类型]]
- ## 十一、[[高阶函数]]
- ## 十二、[[匿名函数]]