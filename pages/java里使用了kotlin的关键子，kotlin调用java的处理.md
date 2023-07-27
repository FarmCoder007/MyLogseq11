- ## java
	- ```java
	  public class JavaClass {
	      public static String in = "123";
	  }
	  
	  ```
	- in 为 kotlin的关键字，
- ## kotlin 处理
	- 会自动加上  ` `
	- ```kotlin
	  class MainKotlin {
	  
	      fun test(){
	          JavaClass.`in`
	      }
	  }
	  ```