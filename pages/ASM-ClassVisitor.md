- 一、
- ## 二、相关api
- ## 三、api调用顺序，从上到下
	- 1、visit( ) ：访问类的开始的部分。
		- ```java
		  // version:类版本。次要版本存储在16个最高有效位中，主要版本存储在十六个最低有效位中
		  // access: 类的访问标志（参见Opcodes）。此参数还指示该类是否已弃用。
		  public void visit(
		        final int version,
		        final int access,
		        final String name,
		        final String signature,
		        final String superName,
		        final String[] interfaces) 
		  ```