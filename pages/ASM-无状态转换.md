- ## 应用场景：查找方法，替换方法
- ## 简介：
	- 转换是局部的，不会依赖于在当前指令之前访问的指令
- ## 场景一、查找方法
	- 给定一个方法包括（类信息，方法名及方法描述符）查找所有调用的地方，输出类和方法集合
	- ### step1、创建自定义MethodFindRefVisitor继承自ClassVisitor,重写visitMethod()判断code中是否调用了指定查找信息
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
	- Java虚拟机规范中定义如下：
	  collapsed:: true
		- invokespecial: 在操作数栈中obj之后必须跟随n个参数值，他们的数量，数据类型和顺序都必须与方法描述符保持一致，若调用不是本地方法，n个参数值和obj将从操作数栈中出栈，方法调用时，将在java虚拟机栈中创建一个新的栈桢，obj和连续的n个参数值将存储到新栈桢的局部变量表中，obj存储到局部变量表0，n个参数依次往后，新栈桢创建后就成为当前栈桢，JVM的PC寄存器指向待调用方法中首条指令，程序就从这里开始继续执行；
		- invokestatic: 在操作数栈中必须包含连续n个参数值，这些参数的数量，数据类型和顺序都必须遵循实例方法的描述符，若调用不是本地方法，n个参数值将从操作数栈中出栈，方法调用时，将在java虚拟机栈中创建一个新的栈桢，连续的n个参数值将存储到新栈桢的局部变量表中，新栈桢创建后就成为当前栈桢，JVM的PC寄存器指向待调用方法中首条指令，程序就从这里开始继续执行；
	- 代码举例：
		- 代码：
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
		- 字节码指令解析：
		  collapsed:: true
			- invokespecial调用add方法时会将已经加载到操作数栈上的b , a, this依次出栈消耗掉，因此当我们替换该instruction时的方法也是需要消耗掉操作数栈上的 b, a, this数据，否则后续执行可能会报错；
			- invokestatic调用之前只执行了ldc将常量池中3位置加载到操作数栈后执行getDesc方法，只消耗了该String对象
			- 使用工具类[AnalyzerAdapter](https://javadoc.io/static/org.ow2.asm/asm-commons/9.3/org/objectweb/asm/commons/AnalyzerAdapter.html) [源码](https://github.com/killme2008/aviatorscript/blob/master/src/main/java/com/googlecode/aviator/asm/commons/AnalyzerAdapter.java)对上方test方法的每行Instruction执行时栈桢中局部变量表和操作数栈数据如下
			  collapsed:: true
				- ```java
				  test:(II)V			 局部变量表         ｜  操作数栈
				                                 // {this, int, int} | {}
				  0000: aload_0                  // {this, int, int} | {this}
				  0001: iload_1                  // {this, int, int} | {this, int}
				  0002: iload_2                  // {this, int, int} | {this, int, int} 
				  0003: invokespecial   #2       // {this, int, int} | {int} //将操作数栈上数据消耗掉后存储返回值
				  0006: pop                      // {this, int, int} | {}
				  0007: ldc             #3       // {this, int, int} | {String}
				  0009: invokestatic    #4       // {this, int, int} | {String} //消耗掉String后存储返回值
				  0012: pop                      // {this, int, int} | {}
				  0013: return                   // {} | {}
				  
				  //单例模式的字节码
				  0: getstatic     #2                  // Field instance:Lsample/HelloWorld;
				  3: ifnonnull     16
				  6: new           #3                  // class sample/HelloWorld
				  9: dup
				  10: invokespecial #4                  // Method "<init>":()V
				  13: putstatic     #2                  // Field instance:Lsample/HelloWorld;
				  16: getstatic     #2                  // Field instance:Lsample/HelloWorld;
				  19: areturn
				  
				  getInstance()Lsample/HelloWorld;
				  [] []
				  [] [sample/HelloWorld]
				  [] []
				  [] [uninitialized_sample/HelloWorld]
				  [] [uninitialized_sample/HelloWorld, uninitialized_sample/HelloWorld]
				  [] [sample/HelloWorld]
				  [] []
				  [] [sample/HelloWorld]
				  [] []
				  ```
	- ## 通过以上观察，对方法替换分为静态方法和非静态方法
	  collapsed:: true
		- 代码：
		  collapsed:: true
			- ```java
			  //自定义MethodReplaceInvokeAdapter extends MethodVisitor替换visitMethodInsn方法
			  public void visitMethodInsn(int opcode, String owner, String name, String desc, boolean itf) {
			      if (oldOwner.equals(owner) && oldMethodName.equals(name) && oldMethodDesc.equals(desc)){
			      	//这里是我们自定义替换的方法
			          super.visitMethodInsn(newOpcode , newOwner , newMethodName , newMethodDesc , false);
			      }else{
			          super.visitMethodInsn(opcode, owner, name, desc, itf);
			      }
			  }
			  
			  /**
			  * 调用区别newMethodDesc方法描述符是否需要消耗掉this
			  */
			  //静态方法替换，描述符中不添加this
			   ClassVisitor cv = new MethodReplaceInvokeVisitor(Opcodes.ASM9, cw,
			                  "sample/HelloWorld", "getDesc", "(Ljava/lang/String;)Ljava/lang/String;",
			                  Opcodes.INVOKESTATIC, "com/asm/method/ReplaceMethodManager", "getDesc", "(Ljava/lang/String;)Ljava/lang/String;"); 
			  //非静态方法替换
			  ClassVisitor cv = new MethodReplaceInvokeVisitor(Opcodes.ASM9, cw, "sample/HelloWorld", "add", "(II)I", Opcodes.INVOKESTATIC, "com/asm/method/ReplaceMethodManager", "add", "(Lsample/HelloWorld;II)I");
			  
			  ```
		- 由于替换方法由我们定义，一般使用的是静态方法newOpcode设为Opcodes.INVOKESTATIC，最后一个参数是boolean isInterface，若为true表示调用为一个接口里的方法，若为false表示调用为一个类中方法，由于操作我们拥有自主权，一般写成在一个类里的static方法