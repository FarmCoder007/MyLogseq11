- ## 一、函数声明  fun关键字
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
- ## 二、[[kotlin  扩展属性]]
- ## 三、inline关键字   被称为 内联函数
  collapsed:: true
	- 使用inline关键字声明的函数是「内联函数」，在「编译时」会将「内联函数」中的函数体直接插入到调用处。所以在写内联函数的时候需要注意，尽量将内联函数中的代码行数减少！
	-
	- ## 作用 ：
		- 一小步优点：    减少一层函数调用栈     【意义不大】
		- 最大优点 ： 在接收类型是函数类型的方法 时
			- 不声明成内联函数  每次调用都会生成一个对象 ，
			- 而对于传入参数是函数类型的函数 最大的优点是减少对象的产生
	- 注意：  尽量将内联函数中的代码行数减少
	- ## 使用场景：
		- 内联函数  适合 传入参数类型  是函数类型的参数
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
- ## 六、kotlin标准函数
  collapsed:: true
	- 使⽤时可以通过简单的规则作出⼀些判断：
	- 返回⾃身 -> 从 apply 和 also 中选
	- 作⽤域中使⽤ this 作为参数 ----> 选择 apply
	- 作⽤域中使⽤ it 作为参数 ----> 选择 also
	- 不需要返回⾃身 -> 从 run 和 let 中选择
	- 作⽤域中使⽤ this 作为参数 ----> 选择 run
	- 作⽤域中使⽤ it 作为参数 ----> 选择 let
	- apply 适合对⼀个对象做附加操作的时候
	- with 适合对同⼀个对象进⾏多次操作的时候
	- apply 做一些附加操作 示例：
	  collapsed:: true
		- ```java
		  // 正常的初始化操作  
		      // 声明变量
		      val paint = Paint()
		      // 再初始化
		      init {
		          paint.isAntiAlias = true
		          paint.strokeWidth = 2f
		      }
		   
		   
		  // 通过apply操作符     this  也可以省略
		      val paint = Paint().apply {
		          this.isAntiAlias = true
		          this.strokeWidth = 2f
		      }
		  ```
	- let 适合配合空判断的时候 ( 最好是成员变量，⽽不是局部变量，局部变量更适合⽤ if )
- ## 七、静态函数   kotlin没有static 关键字     三种方法实现 静态函数
  collapsed:: true
	- ## 方法一 ： 将静态变量 或者 静态函数 写在 kotlin 文件里 而不是类里
	  collapsed:: true
		- kotlin会把这些 自动生成 属于这个类的静态变量  或者 静态函数
		- ```java
		  // utils.kt  文件里声明函数          这种称为包级函数 
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
		   
		   
		  //1 这样在kotlin 文件里直接可调用
		  //2.1 在 java文件里  也可直接调用   需要倒下包
		  import static com.lazy.kotlinapplication.learnkotlin.FunKt.doubleNumber;
		   
		  public class MainActivity extends AppCompatActivity implements View.OnClickListener {
		      @Override
		      protected void onCreate(Bundle savedInstanceState) {
		          super.onCreate(savedInstanceState);
		          setContentView(R.layout.activity_main);
		      }
		   
		      @Override
		      public void onClick(View v) {
		          doubleNumber(2);
		      }
		  }
		  // 2.2 kotlin文件名Kt.函数名调用
		  import com.lazy.kotlinapplication.learnkotlin.FunKt;
		   
		  // 调用
		  FunKt.doubleNumber(2);
		   
		  // 这个FunKt  这个文件名Kt  可以通过注解修改的   @file:JvmName("kotlinUtils")    file是针对文件的
		   
		  @file:JvmName("kotlinUtils")
		  package com.lazy.kotlinapplication.learnkotlin
		   
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
		   
		  //调用
		  import com.lazy.kotlinapplication.learnkotlin.kotlinUtils;
		  kotlinUtils.doubleNumber(2);
		  ```
	- ## 方式二：使用object  声明kotlin 类
	  collapsed:: true
		- 这个类会自动创建一个单例对象   就可以直接调用   成员变量 方法 属性
		- 调用时  会使用   类名,方法名   调用
		- ```java
		  // kotlin  使用 object声明类
		  object ToastUtils {
		      fun toast(context: Context, str: String?) {
		          Toast.makeText(context, str, Toast.LENGTH_LONG).show();
		      }
		  }
		   
		  // kotlin调用      直接类名.方法名
		  ToastUtils.toast(this,"Kotlin 调用")
		   
		  // java里调用   
		   ToastUtils.INSTANCE.toast(this,"java里调用 需要用INSTANCE实例");
		  ```
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
- ## 十 、[lambda 表达式](https://www.cnblogs.com/Jetictors/p/8647888.html)，定义函数
  collapsed:: true
	- 示例  ：循环遍历 使用lambad
	- ```kotlin
	  var strList = mutableListOf("22", "33", "55")
	   // 正常遍历
	      for (str in strList) {
	          println(str)
	      }
	      // 使用lambda
	      // lambda  1 声明在{}里
	      //         2 des: String -> println(des)      格式 参数名：参数类型 ->  函数体
	      strList.forEach({
	          des: String -> println(des)
	      })
	      // 注1： 在kotlin中  如果函数 最后一个参数是lambda  需要把lambda移到最外边
	      strList.forEach(){
	          des: String -> println(des)
	      }
	      // 注2： 在kotlin中  如果函数传入参数 只有一个lambda的话  （）可以省略
	      strList.forEach{
	          des: String -> println(des)
	      }
	      // 由于kotlin 有类型推断  遍历的是string集合  类型可以省略
	      strList.forEach{
	          des -> println(des)
	      }
	      // 注3 ： lambda只有一个传入参数的话  参数可以省略  用隐式的it来访问这个参数
	      strList.forEach{
	         println(it)
	      }
	  ```