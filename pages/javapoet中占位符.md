- $L
- $N
- $S
- $T
	- $T 在JavaPoet代指的是TypeName，该模板主要将Class抽象出来，用传入的TypeName指向的Class来代替
	- 例子：
		- ```java
		  ClassName bundle = ClassName.get("android.os", "Bundle");
		  addStatement("$T bundle = new $T()",bundle)
		  
		    
		    
		    
		  ```
- $$
- $W
- $Z
- $>
- $<
- $[
- $]