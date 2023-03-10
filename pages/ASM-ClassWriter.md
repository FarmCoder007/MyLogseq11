- ## 一、作用
	- 将各个不同的部分重新组合成一个完整的.class文件，如何实现呢？
- ## 二、源码实现
	- ### 构造函数解析
	  collapsed:: true
		- 构建代码：
		  collapsed:: true
			- ```java
			  public class ClassWriter extends ClassVisitor {
			      public static final int COMPUTE_MAXS = 1;
			      public static final int COMPUTE_FRAMES = 2;
			      
			      private int version; //版本号
			      private final SymbolTable symbolTable; //常量池信息
			       
			      private int accessFlags;  //标识位
			      private int thisClass;  //当前类索引
			      private int superClass; //当前类父类索引
			      private int interfaceCount; //接口数据
			      private int[] interfaces;
			      
			      private FieldWriter firstField; //字段表
			      private FieldWriter lastField;
			      
			      private MethodWriter firstMethod; //方法表
			      private MethodWriter lastMethod;
			      
			      private Attribute firstAttribute; //属性表
			      
			      //通过构造函数封装为SymbolTable对象：主要是解析类信息中，主要是常量池信息
			      public ClassWriter(ClassReader classReader, int flags) {
			          super(589824);
			          this.symbolTable = classReader == null ? new SymbolTable(this) : new SymbolTable(this, classReader);
			      }
			      
			      public final void visit(int version, int access, String name, String signature, String superName, String[] interfaces) {
			          this.version = version;
			          this.accessFlags = access;
			          //根据类名全限定名获取在常量池中下标
			          this.thisClass = this.symbolTable.setMajorVersionAndClassName(version & \'uffff\', name);
			          if (signature != null) {
			              this.signatureIndex = this.symbolTable.addConstantUtf8(signature);
			          }
			  			
			          this.superClass = superName == null ? 0 : this.symbolTable.addConstantClass(superName).index;
			          if (interfaces != null && interfaces.length > 0) {
			              this.interfaceCount = interfaces.length;
			              this.interfaces = new int[this.interfaceCount];
			  
			              for(int i = 0; i < this.interfaceCount; ++i) {
			                  this.interfaces[i] = this.symbolTable.addConstantClass(interfaces[i]).index;
			              }
			          }
			      }
			      //字段及方法通过链表连接，由firstField -> lastField; firstMethod -> lastMethod
			       public final FieldVisitor visitField(int access, String name, String descriptor, String signature, Object value) {
			          FieldWriter fieldWriter = new FieldWriter(this.symbolTable, access, name, descriptor, signature, value);
			          if (this.firstField == null) {
			              this.firstField = fieldWriter;
			          } else {
			              this.lastField.fv = fieldWriter;
			          }
			          return this.lastField = fieldWriter;
			      }
			  
			      public final MethodVisitor visitMethod(int access, String name, String descriptor, String signature, String[] exceptions) {
			          ...
			      }
			      //属性表也是通过链表
			      public final void visitAttribute(Attribute attribute) {
			          attribute.nextAttribute = this.firstAttribute;
			          this.firstAttribute = attribute;
			      }
			  }
			  ```
		- 原理：
			- ClassWriter通过构造函数将传入的ClassVisitor信息解析封装为SymbolTable对象并将用到的classFile中数据保存为全局变量，字段field，method，Attribute等数据均由链表表示；
	- ### toByteArray函数——>组装.class文件
		- 1、计算byte[]数组，即class文件大小size；[[计算方式在toByteArray() 函数]]
		- 2、向byte数组中按照classFile格式添加对应元素；[[添加数据toByteArray() 函数分析]]
		- 3、将byte[] 数据返回;
-
-