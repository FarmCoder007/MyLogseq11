- ## 应用场景：查找方法，替换方法
- ## 简介：
	- 转换是局部的，不会依赖于在当前指令之前访问的指令
- ## 场景一、查找方法
	- 给定一个方法包括（类信息，方法名及方法描述符）查找所有调用的地方，输出类和方法集合
	- ### step1、创建自定义MethodFindRefVisitor继承自ClassVisitor,重写visitMethod()判断code中是否调用了指定查找信息
	  collapsed:: true
		- ```java
		  @Override
		  public MethodVisitor visitMethod(int access, String name, String descriptor, String signature, String[] exceptions) {
		      boolean isAbstractMethod = (access & ACC_ABSTRACT) != 0;
		      boolean isNativeMethod = (access & ACC_NATIVE) != 0;
		      if (!isAbstractMethod && !isNativeMethod) {
		          return new MethodFindRefAdaptor(api, null, owner, name, descriptor);
		      }
		      return null;
		  }
		  ```
	- ### ste