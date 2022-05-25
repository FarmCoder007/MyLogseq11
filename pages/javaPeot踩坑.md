- 1、方法体中处理类导包问题
	- java代码：直接使用$T  第二个参数传指定类的ClassName类型
		- ```
		  methodBuilder.addStatement("$T substitute = ($T)target", 类的ClassName);
		  ```
	- kotlin代码写的Process $会被kt认为变量引用，需要转义符\$T调用
		- ```
		  .addStatement("Object object = \$T.navigation(context, \"${autoGenerationAnnotationImpl.routerPath}\")",WBRouterClassNameJAVA)
		  ```
- 2、解析kotlin写的