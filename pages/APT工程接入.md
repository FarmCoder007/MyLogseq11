- # 一、工程创建
	- ## 1、创建annotation模块->一定是java library[存放注解]
		- File -> New Module -> Java library
		- ### 1-1、新建注解
			- ```
			  @Retention(RetentionPolicy.CLASS) //表示编译时的注解
			  @Target(ElementType.TYPE) //表示作用在变量上
			  public @interface AutoGenerationAnnotation {
			      // 注解参数
			      String routerPath();
			  }
			  ```
		- ### 1-2、使用注解
			- ```
			  
			  ```
		-
		-
	- ## 2、创建processor模块->一定是java library[解析处理注解]
	-
	-
-