- ## 定义callBack
	- ## java
	  collapsed:: true
		- JavaCallback
		- ``` java
		  public interface JavaCallback {
		  	void show(String info);
		  }
		  ```
		- JavaManager 设置callBack
			- ```java
			  public class JavaManager {
			  
			      public void setCallback(JavaCallback javaCallback) {
			          javaCallback.show("Derry1");
			      }
			  
			  }
			  ```
	- ## kotlin
		- KTCallback
		  collapsed:: true
			- ```kotlin
			  interface KTCallback {
			  
			      fun show(name: String)
			  
			  }
			  ```
		- KtManager
		  collapsed:: true
			- ```kotlin
			  class KtManager {
			  
			      fun setCallback(callback: KTCallback) {
			          callback.show("Kt Derry")
			      }
			  
			  }
			  ```
- ## kt使用java cb
  collapsed:: true
	- ```kotlin
	  fun main() {
	  
	      // TODO kt 使用 Java Callback
	      // 第一种写法
	      JavaManager().setCallback(JavaCallback {
	          println(it)
	      })
	  
	      // 第二种写法
	      JavaManager().setCallback(object : JavaCallback{
	          override fun show(info: String?) {
	              println(info)
	          }
	      })
	  
	      // 第三种写法  callback 提出来
	      val callback = JavaCallback {
	          println(it)
	      }
	      JavaManager().setCallback(callback)
	  
	      // 第四种写法
	      val callback2 = object : JavaCallback{
	          override fun show(info: String?) {
	              println(info)
	          }
	      }
	      JavaManager().setCallback(callback2)
	  }
	  ```
- ## kt 使用kt cb
	- ```kotlin 
	   // TODO kt 使用 kt Callback
	      // 1
	      KtManager().setCallback(object : KTCallback{
	          override fun show(name: String) {
	          }
	      })
	  
	      // 2
	      val c = object : KTCallback{
	          override fun show(name: String) {  }
	      }
	      KtManager().setCallback(c)
	  
	      // Kt不能像Java一样的写法
	      /*KtManager().setCallback(KTCallback{
	  
	      })*/
	  ```