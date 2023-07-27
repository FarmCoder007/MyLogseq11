- kotlin里定义方法,:kotlin  KClass  和 java 类的class
	- ```kotlin 
	  class MainKotlin {
	  
	      fun test(){
	          var string:String? = JavaClass().name
	  
	          showClass(Student::class.java)
	          showClass2(KtStudent::class)
	      }
	  
	  
	  
	      // 形参 为java 的类
	      fun showClass(clazz:Class<Student>){
	  
	      }
	  
	      // 形參 为kotlin  则用KClass
	      fun showClass2(clazz:KClass<KtStudent>){
	  
	      }
	  
	  }
	  ```
- Student为java类
	- ```java
	  package com.example.myapplication.ss;
	  
	  public class Student {
	  }
	  
	  ```
- KtStudent为kotlin类
	- ```java
	  package com.example.myapplication.ss
	  
	  class KtStudent {
	  }
	  ```