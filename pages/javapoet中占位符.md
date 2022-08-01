- $L
- $N
	- $N 在JavaPoet中代指的是一个名称，例如调用的方法名称，变量名称，这一类存在意思的名称
	- 例子：
		- ```java
		  addStatement("data.$N()",toString)
		  // 即调用data中的方法
		  data.toString();
		    
		  ```
- $S
	- \$S 在JavaPoet中就和 String.format 中 %s 一样,字符串的模板,将指定的字符串替换到$S的地方
	- 例子：
		- ```java
		  .addStatement("super.$S(savedInstanceState)","onCreate")
		  ```
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
	- $$发出美元符号。
- $W
	- $W根据其在行的位置发出空格或换行
- $Z
- $>
	- $>增加缩进级别
- $<
	- $<降低缩进级别。
- $[
	-
- $]