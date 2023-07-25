- # 一、[[kotlin.String]]
- # 四、kotlin 空安全问题
  collapsed:: true
	- ### 1、声明   kotlin严格 区分 变量 可空 与 不可空（可null  和 不可为null）
		- 变量类型后 用？修饰  代表 可空 变量      没有修饰 即为 非空变量
		- ```java
		  // 1 正常定义的变量   没有？修饰就是非空变量
		  // kotlin 成员变量 必须初始化 或者定义抽象
		  private val usernameKey: String = "username"
		  // 2 不想初始化 赋值null  必须？修饰  代表 可为空变量
		   private val usernameKey: String？ = null
		  // 这种非空变量  不被允许赋值null 
		   private val usernameKey: String = null
		  ```
	- ### 2、？对应 java里的@Nullable    非空 对应@NonNull
		- java空安全问题 ：
			- 1 注解 @NonNull  @Nullable  这些 了解少的不会用
			- 2 即使加上注解有提示报错  但是 依然可以编译通过
			- kotlin 里 如果有变量可能为null  必须用？ 声明 为可空类型  要不然会报错
	- ### 3、对于定义 ？可空变量  执行问题
	  collapsed:: true
		- ```java
		  // 定义可为null的变量 et_name
		  private var et_name: EditText? = null
		  // 未真正初始化  调用可空 变量 下边.会报错
		  et_name.setText("")
		   
		  // 可空变量调用方法  2种方案
		  // 1 变量后？修饰    先会判断et_name是否为null  为null  就不会调用
		  et_name？.setText("")
		  // 2 变量后！！修饰   强制执行  不管是否为null  为null就会报空指针
		  et_name!!.setText("")
		  ```
	- ### 4、对于控件的初始化问题   lateinit  延迟初始化关键字
		- ## 背景
		  collapsed:: true
			- 1 声明变量必须初始化
			- 2 控件初始化必须在 setContentView(R.layout.activity_main)  之后才能findViewById
			- 3 如果 先使用  可空修饰符？修饰的话
			- ```java
			   private var et_name: EditText? = null
			   
			   override fun onCreate(savedInstanceState: Bundle?) {
			          super.onCreate(savedInstanceState)
			          setContentView(R.layout.activity_main)
			          et_name = findViewById(R.id.et_name);
			          // 这种写法 万一 layout设置错了   下边这句不会执行 不会报错    运行后text不会变
			          // 排查问题难  还不如直接报空    lateinit延迟初始化 解决
			          et_name?.setText("测试");
			   }
			  ```
		- ## [[lateinit]]
			-
- # 五、平台类型
  collapsed:: true
	- ```java
	  声明成员变量 且未初始化值 需要带变量类型
	  private lateinit var et_name: EditText
	   
	  局部变量 通过findViewById  初始化变量时   由于kotlin 会检测非kotlin变量 会检测到平台类型
	  // 可不加变量类型 button是java平台的  会隐藏 一个 平台类型
	  val bt_button = findViewById<Button>(R.id.btn_one)
	  ```
- # 八、kotlin  自定义view 等 调用this  或super方法
  collapsed:: true
	- ```java
	  // java
	  public class MyImageView extends AppCompatImageView {
	      public MyImageView(Context context) {
	          this(context, null);
	      }
	   
	      public MyImageView(Context context, AttributeSet attrs) {
	          super(context, attrs);
	      }
	  }
	   
	   
	  //kotlin   在构造函数后调用
	  class MyImageView : AppCompatImageView {
	      constructor(context: Context) : this(context, null){
	   
	      }
	   
	      constructor(context: Context, attr: AttributeSet?) : super(context, attr){
	   
	      }
	  }
	  ```
- # 九、数组的声明
  collapsed:: true
	- ```java
	      /**
	       *  数据类型
	       *  java  基本数据类型  int float double long   （声明的是值）     不可空的类型
	       *        对应的是下边
	       *  kotlin 基本数据类型  Int  Float Double Long    （kotlin里这些是对象的 带有相应方法的 ）
	       *         比如1.toFloat()    kotlin 基本数据类型是带方法的
	       *
	       *  java   包装类型     Int  Float Double Long       这些是可空的类型
	       *  对应kotlin 可空类型 Int?  Float?
	       */
	  ```
	- ```kotlin
	  // java  所有的都是   数组类型[]声明
	   private String[] strArray = new String[]{"44","33","22","11"};
	   
	  // kotlin   对于非基本数据类型  使用arrayOf()  直接声明
	  private var strArray = arrayOf("1","2","3")
	   
	   
	  // 对于基本数据类型  使用arrayOf  对自动装箱操作   取的时候拆箱   耗费内存  所以使用基本数据类型专门的生成方式
	  private var intArray = intArrayOf(1,2,3)
	  private var s = floatArrayOf(1f,2f,2f)
	  private var bo = booleanArrayOf(false,true)      等等
	  ```
-