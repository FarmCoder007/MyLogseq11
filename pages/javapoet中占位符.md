- $L
- $N
	- $N 在JavaPoet中代指的是一个名称，例如调用的方法名称，变量名称，这一类存在意思的名称
	- 例子：
		- ```java
		  addStatement("data.$N()",toString)
		  
		    
		  ```
- $S
- $T
	- $T 在JavaPoet代指的是TypeName，该模板主要将Class抽象出来，用传入的TypeName指向的Class来代替
	- 例子：
		- ```java
		  ClassName bundle = ClassName.get("android.os", "Bundle");
		  addStatement("$T bundle = new $T()",bundle)
		  
		  // 生成方法
		  Bundle bundle = new Bundle();
		  ```
- $$
- $W
- $Z
- $>
- $<
- $[
- $]