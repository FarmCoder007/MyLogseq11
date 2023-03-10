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
- ## 场景二、替换方法
	- 若想替换掉某一条instruction，应该如何实现呢？当然是首先找到该instruction，然后在同样的位置替换成另一个instruction就可以啦！
	- 需要特别注意： 在替换instruction过程中，operand stack(栈桢数据)在修改前和修改后是一致的
	- 代码举例：
	  collapsed:: true
		- ```java
		  // java源码
		  public class HelloWorld {
		      private HelloWorld(){}
		      private static HelloWorld instance ;
		      public static synchronized HelloWorld getInstance(){
		          if (instance == null){
		              instance = new HelloWorld();
		          }
		          return instance;
		      }
		  
		      public void test(int a, int b){
		          add(a , b); //调用非静态方法
		          getDesc("HelloWorld"); //调用静态方法
		      }
		      private int add(int a , int b){
		          return a + b;
		      }
		      public static String getDesc(String clazzName){
		          return "current class " + clazzName;
		      }
		  }
		  
		  // 输出字节码如下
		  public void test(int, int);
		      descriptor: (II)V
		      flags: (0x0001) ACC_PUBLIC
		      Code:
		        stack=3, locals=3, args_size=3  //局部变量表中位置： 0:this； 1:a ; 2:b
		           0: aload_0   //加载this到操作数栈
		           1: iload_1   //加载a到操作数栈
		           2: iload_2   //加载b到操作数栈
		           3: invokespecial #2                  // Method add:(II)I  调用非静态方法add
		           6: pop
		           7: ldc           #3                  // String HelloWorld
		           9: invokestatic  #4                  // Method getDesc:(Ljava/lang/String;)Ljava/lang/String;
		          12: pop
		          13: return
		          
		  //字节码 对应的 asm代码如下
		  {
		    methodVisitor = classWriter.visitMethod(ACC_PUBLIC, "test", "(II)V", null, null);
		    methodVisitor.visitCode();
		    methodVisitor.visitVarInsn(ALOAD, 0);
		    methodVisitor.visitVarInsn(ILOAD, 1);
		    methodVisitor.visitVarInsn(ILOAD, 2);
		    methodVisitor.visitMethodInsn(INVOKESPECIAL, "sample/HelloWorld", "add", "(II)I", false); //调用non-static add方法，上面三个visitVarInsn方法是方法所需参数
		    methodVisitor.visitInsn(POP);
		    
		    methodVisitor.visitLdcInsn("HelloWorld");
		    methodVisitor.visitMethodInsn(INVOKESTATIC, "sample/HelloWorld", "getDesc", "(Ljava/lang/String;)Ljava/lang/String;", false); //调用static getDesc需要一个参数
		    methodVisitor.visitInsn(POP);
		    methodVisitor.visitInsn(RETURN);
		    methodVisitor.visitMaxs(3, 3);
		    methodVisitor.visitEnd();
		  }
		  ```
	- Java虚拟机规范中定义如下：
		-