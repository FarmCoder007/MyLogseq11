- ## 一、将变量和函数写在kotlin 文件里（非类）
	- kotlin会把这些 自动生成 属于这个类的静态变量  或者 静态函数
- # 二、示例
	- kotlin文件名：MyUtils.kt
		- ```kotlin
		  package com.xiangxue.rxjavademo.demo
		  
		  
		  fun doubleNumber(x: Int): Int {
		      return x * 2
		  }
		  
		  ```
	- 反编译成java代码
		- ```java
		  package com.xiangxue.rxjavademo.demo;
		  
		  import kotlin.Metadata;
		  
		  @Metadata(
		     mv = {1, 8, 0},
		     k = 2,
		     d1 = {"\u0000\n\n\u0000\n\u0002\u0010\b\n\u0002\b\u0002\u001a\u000e\u0010\u0000\u001a\u00020\u00012\u0006\u0010\u0002\u001a\u00020\u0001¨\u0006\u0003"},
		     d2 = {"doubleNumber", "", "x", "RxJavaDemo.app.main"}
		  )
		  public final class MyUtilsKt {
		     public static final int doubleNumber(int x) {
		        return x * 2;
		     }
		  }
		  
		  ```
		- 可以看到类名是拼接的：文件名+Kt：MyUtilsKt
		- 内部生成静态方法
	- ## 调用上的区别
		- kotlin类里可以直接调用
			- MainKotlin.kt
			- ```java
			  package com.xiangxue.rxjavademo.demo
			  
			  class MainKotlin {
			  
			      fun test(){
			          doubleNumber(1)
			      }
			  }
			  ```
		- java里需要用生成的文件名调用比如MyUtilsKt.doubleNumber
			- ```java
			      public void testKotlin(){
			          MyUtilsKt.doubleNumber();
			      }
			  ```
		- 可以修改这个生成的文件名字MyUtilsKt   @file:JvmName("kotlinUtils")，修饰包名
			- ```java
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
-