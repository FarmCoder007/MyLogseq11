- # 1、单参构造函数
	- 场景：如果View是在Java代码里面new的，则调用第一个构造函数
	- ```java
	      public CarsonView(Context context) {
	          super(context);
	      }
	  ```
- # 2、双参
	- 场景：如果View是在.xml里声明的，则调用第二个构造函数
	- ```java
	      // 自定义属性是从AttributeSet参数传进来的
	      public CarsonView(Context context, AttributeSet attrs) {
	          super(context, attrs);
	      }
	  ```
- # 3、三参数
	- 场景：不会自动调用，一般是在第二个构造函数里主动调用，有主题 style时
	- ```java
	      // 不会自动调用
	      // 一般是在第二个构造函数里主动调用
	      // 如View有style属性时
	      public CarsonView(Context context, AttributeSet attrs, int defStyleAttr) {
	          super(context, attrs, defStyleAttr);
	      }
	  ```
- # 4、四个参数
	- 场景：一般是在第二个构造函数里主动调用
	- ```java
	  
	     //API21之后才使用
	      // 不会自动调用
	      // 一般是在第二个构造函数里主动调用
	      // 如View有style属性时
	      public CarsonView(Context context, AttributeSet attrs, int defStyleAttr, int
	              defStyleRes) {
	          super(context, attrs, defStyleAttr, defStyleRes);
	      }
	  
	  ```