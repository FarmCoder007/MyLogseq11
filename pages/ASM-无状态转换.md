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
	- ### step2、创建自定义MethodFindRefAdaptor继承自MethodVisitor，对方法体code进行解析重写visitMethodInsn判断是否调用查找方法
	  collapsed:: true
		- ```java
		  @Override
		  public void visitMethodInsn(int opcode, String owner, String name, String descriptor, boolean isInterface) {
		      // 首先，处理自己的代码逻辑，判断当前方法体中调用了指定查找的方法，则存储当前类和方法信息
		      if (methodOwner.equals(owner) && methodName.equals(name) && methodDesc.equals(descriptor)) {
		          String info = String.format("%s.%s%s", currentMethodOwner, currentMethodName, currentMethodDesc);
		          if (!resultList.contains(info)) {
		          resultList.add(info);
		          }
		      }
		  
		      // 其次，调用父类的方法实现
		      super.visitMethodInsn(opcode, owner, name, descriptor, isInterface);
		  }
		  ```
	- ### step3、在合适时机对获取数据解析输出打印或文件中
- ## 场景二、