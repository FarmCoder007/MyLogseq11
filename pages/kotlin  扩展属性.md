- 1 扩展函数可以为任何类添加上一个函数，从而代替工具类
- 2 扩展函数和成员函数相同时，成员函数优先被调用   不用怕把类的方法覆盖
- 3 扩展函数是静态解析的，在编译时就确定了调用函数(没有多态)
- 示例1
	- ```java
	  // kotlin 定义 一个 dp2px的工具类  
	  private val displayMetrics = Resources.getSystem().displayMetrics
	   
	  fun dp2px(dp: Float): Float {
	      return TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, dp, displayMetrics)
	  }
	   
	  // 使用的时候
	  dp2px(16f)
	   
	  ```
- 为了提高可读性  想给 Float 类 扩展个dp2px方法   那么在方法名前 加 要扩展的类名.
	- ```java
	   
	   
	  // 1 下边为Float 类扩展函数   在dp2px 前加 Float.  
	  // 2 去掉输入的自己类型的参数    函数里参数用this代替自己
	  fun Float.dp2px(): Float {
	      return TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, this, displayMetrics)
	  }
	   
	  //调用
	  16f.dp2px()
	  ```