- ## 一、object
  collapsed:: true
	- ## 1、用于匿名内部类声明
		- ```kotlin
		  //  Proxy 创建代理类的方法
		  public static Object newProxyInstance(ClassLoader loader,
		                                          Class<?>[] interfaces,
		                                           InvocationHandler h)
		  // InvocationHandler匿名内部类传入
		  Proxy.newProxyInstance(classLoader , proxyInterfaces , object :InvocationHandler{
		                      override fun invoke(proxy: Any?, method: Method?, args: Array<out Any>?): Any {
		                          TODO("Not yet implemented")
		                      }
		                  })
		  ```
	- ## 2、用于伴生对象声明
		- ```kotlin
		  class CompanionOuter {
		      companion object CompanionInner {
		          fun companionPrint() {
		              println("companionPrint")
		          }
		      }
		  }
		  
		  ```
		- 伴生对象对应的java代码:
		  collapsed:: true
			- ```java
			  public final class CompanionOuter {
			     @NotNull
			     public static final CompanionOuter.CompanionInner CompanionInner = new CompanionOuter.CompanionInner((DefaultConstructorMarker)null);
			  
			     public static final class CompanionInner {
			        public final void companionPrint() {
			           String var1 = "companionPrint";
			           System.out.println(var1);
			        }
			  
			        private CompanionInner() {
			        }
			  
			         // $FF: synthetic method**
			     public CompanionInner(DefaultConstructorMarker $constructor_marker) {
			           this();
			        }
			     }
			  }
			  ```
		- 原理：
			- 伴生对象在java代码中会对应生成一个静态内部类，在外部类中会生成该静态内部类的静态对象，对该伴生对象的调用实际是通过这个静态对象实现的
	- ## 3、用于单例类声明
		- ```kotlin
		  object TestSingleton {
		      fun objDeclarationPrint() {
		          println("objDeclarationPrint")
		      }
		  }
		  ```
- ## 二、return
	- ## 1、return@forEachIndexed 和 return@forEach 代替java中循环的continue
		- java循环与中断：
		  collapsed:: true
			- ```java
			  for (int i = 0; i < list.size(); i++) {}
			  for (int item : list) {}
			  ```
			- java中的中断循环，break 和 continue (终断本次循序)关键字
		- kotlin循环与中断：
		  collapsed:: true
			- ```kotlin
			  forEachIndexed { index, i -> }
			  forEach {it -> }
			  ```
			- return@forEachIndexed 和 return@forEach 代替java中循环的continue
			- return 代替java 中 break
				- break 方式一：使用 return，好处是简单，坏处是大多数时候我们只是想要结束循环，而非结束整个函数
				  id:: 62ea443e-781e-4891-86e7-39b10bfc827b
				- break 方式二：嵌套一层 Lambda 表达式，循环中使用 return@标签的方式实现非局部返回到外层表达式来模拟 break 的效果
					- ```kotlin
					  private fun testFun() {
					      run out@{
					          (1..4).forEach forEach@{
					              if (it % 2 == 0){
					                  Log.e("hzf","it==${it} 我是偶数")
					                  return@out
					              }
					              Log.e("hzf","it==${it} 我是奇数")
					          }
					          Log.e("hzf","我是 run 里面")
					      }
					      Log.e("hzf","我是最外面")
					  }
					  
					  ```
		- [参考资料](https://blog.csdn.net/Nicholas1hzf/article/details/123621523)
- ## 三、[[lateinit]]
- ## 四、is 类型判断
  collapsed:: true
	- is 对应 instanceof         as对应强转类型   但是在kotlin  如下 先用as判断类型    不用像java那样强转  直接认定就是那种类型
	- 类class获取：例如activity跳转  中
		- ```java
		  //kotlin  获取kotlin里的类名  类名::.class
		  //        获取java里的类名    类名::.class.java
		   
		  // java 
		  startActivity(new Intent(this,MainActivity.class));
		  // kotlin
		  startActivity(Intent(this,MainActivity::class.java))
		  ```
		- ```java
		     override fun onClick(v: View?) {
		          //  java  v instanceof ImageView   kotlin  使用 is  代替  instanceof
		          if (v is EditText) {
		              // java  强制类型转换   EditText e = (EditText)v;
		              
		              // 优化点  java 判断完类型  还需要强制转换 成  后边判断的类型   kotlin 不需要
		   
		              // val editText = v as EditText         可直接用
		          }
		      }
		  ```