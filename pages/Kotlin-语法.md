- # 一、kotlin== 和 ===
  collapsed:: true
	- kotlin 中  == 代表是调用 equals 函数  对比的是值     而java中  == 代表对比的是对象的地址
	- kotlin  中 === 代表对比的是对象的地址
	- ```java
	  var user = User("name", "word", "code")
	  var copyUser = user.copy()
	   
	  // 对比数据是否相等
	  fun main() {
	      println(user)
	      println(copyUser)
	      //对比地址   
	      println(copyUser === user) // false
	      // 调用equals 函数
	      println(copyUser == user)  // true
	  }
	  ```
- # 二、Kotlin解构
  collapsed:: true
	- ```java
	  // 模拟请求网络   定义返回类型  数据类
	  data class Response(var code: Int, var message: String, var body: String)
	   
	  /**
	   *  执行网络请求函数  返回Response
	   */
	  fun excuteNet(): Response {
	      return Response(200, "成功", "好")
	  }
	   
	  fun main() {
	      // 在java里 取返回对象里的数据   先获取到返回对象 再挨个取
	      var response: Response = excuteNet()
	      println(response.code.toString() + response.message + response.body)
	      // 在 kotlin  解构   excuteNet().val  回车 选择 Create destructuring declaration(创建解构声明)
	      val (code, message, body) = excuteNet()
	      // 可直接使用  返回的解构声明是与返回数据类  构造参数一一对应的
	      println(code.toString() + message + body)
	  }
	  ```
- # 三、判断一个变量是否为null      ？:
  collapsed:: true
	- 如果前边变量 为null则执行 :后边的操作  不为null  :后边的代码就不执行
	- ```java
	  //java
	  if(a == null){
	     return;
	  }
	  // kotlin   使用 ？： 代替  if  null 的判断
	  a?:return
	  ```
	- ```kotlin
	  // lesson.date 为空时使用默认值
	  val date = lesson.date ?: "日日期待定"
	   
	  // lesson.state 为空时提前返回函数
	  val state = lesson.state ?: return
	   
	  // lesson.content 为空时抛出异常
	  val content = lesson.content ?: throwIllegalArgumentException("content expected")
	  ```
- # 五、filter  操作符
  collapsed:: true
	- ```java
	   // kotlin 自带 filter 操作符  过滤器  传入过滤条件  返回一个新的集合  简化遍历集合添加过滤条件的过程
	   
	      var newList = mutableListOf<String>()
	      strList.forEach{
	          if(it.length > 1){
	              newList.add(it)
	          }
	      }
	   
	      // 过滤的返回新的集合     过滤条件：it.length>1
	      val filter = strList.filter { it.length > 1 }
	  ```
- # 六、kotlin  函数是可以嵌套的    被嵌套的只能在该函数外层函数 里调用
  collapsed:: true
	- ```java
	  Kotlin 中可以在函数中继续声明函数
	  注意点： 1 内部函数可以访问外部函数的参数
	          2  内部函数每次调用都会生成一个函数对象    频繁调用不适合 这个生成方式  影响性能
	          fun  outer(){
	               fun inner(){}
	          }
	  ```
- # 七、设置 变量不可被其他类改变
  collapsed:: true
	- ```java
	  class MyApplication : Application() {
	      companion object {
	          lateinit var currentContext: Context
	              // 设置该变量 set方法不可被其他类改变
	              private set
	      }
	   
	   
	      override fun onCreate() {
	          super.onCreate()
	          currentContext = this
	      }
	  }
	  ```
- # 八、注解使用处目标
  collapsed:: true
	- 当某个元素可能会包含多种内容(例如构造属性，成员属性)，使用注解时可以通过「注解使用处目标」，让注解对目标发生作用，例如@file:、@get:、@set:等
		- ```java
		  // 伴生对象 声明application后
		  class MyApplication : Application() {
		      companion object {
		          lateinit var currentContext: Context
		              // 设置该变量 set方法不可被其他类改变
		              private set
		      }
		   
		   
		      override fun onCreate() {
		          super.onCreate()
		          currentContext = this
		      }
		  }
		  ```
	- kotlin 调用方法:MyApplication.currentContext
	- java里调用以前说过 需要借助
		- ```java
		  MyApplication.Companion.getCurrentContext();
		   
		   
		  // 当kotlin  application  成员变量加注解   @JvmStatic  
		  // Java 里调用 就可以省去 ConPainion
		  MyApplication.getCurrentContext();
		  ```
	- 通过注解使用目标 可以让在java里调用  方法名
	- 注解目标对象   @file  更改名字的对象是文件   @get   @set 更改的目标对象 是set  get方法
		- ```java
		  class MyApplication : Application() {
		      companion object {
		          @JvmStatic
		          // 更改get方法名 为currentContext
		          @get:JvmName("currentContext")
		          lateinit var currentContext: Context
		              // 设置该变量 set方法不可被其他类改变
		              private set
		      }
		   
		   
		      override fun onCreate() {
		          super.onCreate()
		          currentContext = this
		      }
		  }
		  ```
	- 这时候java调用
		- ```java
		  public class MainActivity extends AppCompatActivity {
		   
		      @Override
		      protected void onCreate(Bundle savedInstanceState) {
		          super.onCreate(savedInstanceState);
		          setContentView(R.layout.activity_main);
		          MyApplication.currentContext();
		      }
		  }
		  ```
- # 十、coerceAtLeast    coerceAtMost
	- 作用：一个值最小不能小于某个值    最大不能大于某个值
	- ```java
	  // 是修正 缩放比例   手指缩放比例 最小不能小于smallScale   最大不能大于 bigScale
	              currentScale = currentScale.coerceAtLeast(smallScale).coerceAtMost(bigScale)
	  ```
- # 十一、[[kotlin-闭包]]