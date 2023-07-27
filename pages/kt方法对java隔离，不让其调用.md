- ## kt 使用`  `包裹函数
	- ```kotlin
	  // java可调用
	  fun add(n1: Int, n2: Int) {
	      println("resut:${n1 + n2}")
	  }
	  
	  // java 不可调用，kt都可调用
	  fun `showTest`() {
	      println("showTest")
	  }
	  
	  
	  fun main() {
	      `showTest`()
	      `   `('M')
	      `4325436465375`("Derry")
	  }
	  ```
- ## java
	- ```java
	  public class Client {
	  
	      void test() {
	          KTStudentKt.show("derry1");
	  
	          // Java　无法 调用
	          /*`showTest`();
	      }
	  
	  }
	  ```