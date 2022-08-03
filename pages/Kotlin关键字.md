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
	- return@forEachIndexed 和 return@forEach 代替java中循环的continue
-