- ## 一、object
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
		  class Test {
		      companion object{
		          fun companionPrint() {
		              println("companionPrint")
		          }
		      }
		  }
		  ```
		- 伴生对象对应的java代码
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
	- ## 3、用于单例类声明
- ## 二、return
-