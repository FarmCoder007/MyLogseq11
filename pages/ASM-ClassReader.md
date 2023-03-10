- ## 一、作用
	- 主要负责读取.class文件里的内容，然后拆分成各个不同的部分；如何实现呢？
- ## 二、ClassReader的实现
	- ### 构造函数分析
	  collapsed:: true
		- ```java
		  public class ClassReader {
		      /**
		       * 跳过 Code 属性的标志。 如果设置了此标志，则不会解析也不访问Code属性。
		       * 在 jacoco 中忽略 code 属性值
		       */
		      public static final int SKIP_CODE = 1;
		  
		      //真实的数据部分
		      final byte[] classFileBuffer;
		      //数据的索引信息:标识了classFileBuffer中数据里包含的常量池位置
		      private final int[] cpInfoOffsets;
		  
		      //标记访问标识（access flag）在classFileBuffer中的位置信息
		      public final int header;
		  
		      /**
		       * 构造
		       *
		       * @param classFileBuffer
		       * @param classFileOffset   ：默认为0，起始位置
		       * @param checkClassVersion
		       */
		      ClassReader(final byte[] classFileBuffer, final int classFileOffset, final boolean checkClassVersion) {
		          this.classFileBuffer = classFileBuffer; //class文件数据字节数组
		          this.b = classFileBuffer;
		          // Check the class' major_version. This field is after the magic and minor_version fields, which
		          // use 4 and 2 bytes respectively.
		          if (checkClassVersion && readShort(classFileOffset + 6) > Opcodes.V16) {
		              throw new IllegalArgumentException(
		                      "Unsupported class file major version " + readShort(classFileOffset + 6));
		          }
		          // Create the constant pool arrays. The constant_pool_count field is after the magic,
		          // minor_version and major_version fields, which use 4, 2 and 2 bytes respectively.
		          // //读取第8个字节位置：常量池大小constant_pool_count
		          int constantPoolCount = readUnsignedShort(classFileOffset + 8);
		          cpInfoOffsets = new int[constantPoolCount];
		          constantUtf8Values = new String[constantPoolCount];
		          // Compute the offset of each constant pool entry, as well as a conservative estimate of the
		          // maximum length of the constant pool strings. The first constant pool entry is after the
		          // magic, minor_version, major_version and constant_pool_count fields, which use 4, 2, 2 and 2
		          // bytes respectively.
		          // //当前常量池起始位置：注意ClassFile由1开始，保留0位置用于未指定任何数据
		          int currentCpInfoIndex = 1;
		          //起始偏移量，首位常量池位置：（魔数u4，次版本号u2，主版本号u2，常量池大小u2）
		          int currentCpInfoOffset = classFileOffset + 10; //从第10个字节开始保存常量
		          int currentMaxStringLength = 0;
		          boolean hasBootstrapMethods = false;
		          boolean hasConstantDynamic = false;
		          // The offset of the other entries depend on the total size of all the previous entries.
		          while (currentCpInfoIndex < constantPoolCount) {
		              /**
		               * 常量池的数据：u1:表示当前数据类型
		               * CONSTANT_Utf8_info {
		               *     u1 tag;  // == 1
		               *     u2 length;
		               *     u1 bytes[length];
		               * }
		               */
		              cpInfoOffsets[currentCpInfoIndex++] = currentCpInfoOffset + 1; ////去掉u1数据类型保存常量数据
		              //当前各个常量的偏移量
		              int cpInfoSize;
		              switch (classFileBuffer[currentCpInfoOffset]) {
		                  case Symbol.CONSTANT_FIELDREF_TAG:
		                  case Symbol.CONSTANT_METHODREF_TAG:
		                  case Symbol.CONSTANT_INTERFACE_METHODREF_TAG:
		                  case Symbol.CONSTANT_INTEGER_TAG:
		                  case Symbol.CONSTANT_FLOAT_TAG:
		                  case Symbol.CONSTANT_NAME_AND_TYPE_TAG:
		                      cpInfoSize = 5;
		                      break;
		                  case Symbol.CONSTANT_DYNAMIC_TAG:
		                      cpInfoSize = 5;
		                      hasBootstrapMethods = true;
		                      hasConstantDynamic = true;
		                      break;
		                  case Symbol.CONSTANT_INVOKE_DYNAMIC_TAG:
		                      cpInfoSize = 5;
		                      hasBootstrapMethods = true;
		                      break;
		                  case Symbol.CONSTANT_LONG_TAG:
		                  case Symbol.CONSTANT_DOUBLE_TAG:
		                      cpInfoSize = 9;
		                      currentCpInfoIndex++;
		                      break;
		                  case Symbol.CONSTANT_UTF8_TAG:
		                      //字符串计算字符串长度作为偏移量cpInfoSize
		                      //tag：u1 length:u2 = 3 加上Short位的length表示bytes数组长度的
		                      cpInfoSize = 3 + readUnsignedShort(currentCpInfoOffset + 1);
		                      if (cpInfoSize > currentMaxStringLength) {
		                          // The size in bytes of this CONSTANT_Utf8 structure provides a conservative estimate
		                          // of the length in characters of the corresponding string, and is much cheaper to
		                          // compute than this exact length.
		                          currentMaxStringLength = cpInfoSize;
		                      }
		                      break;
		                  case Symbol.CONSTANT_METHOD_HANDLE_TAG:
		                      cpInfoSize = 4;
		                      break;
		                  case Symbol.CONSTANT_CLASS_TAG:
		                  case Symbol.CONSTANT_STRING_TAG:
		                  case Symbol.CONSTANT_METHOD_TYPE_TAG:
		                  case Symbol.CONSTANT_PACKAGE_TAG:
		                  case Symbol.CONSTANT_MODULE_TAG:
		                      cpInfoSize = 3;
		                      break;
		                  default:
		                      throw new IllegalArgumentException();
		              }
		              currentCpInfoOffset += cpInfoSize;
		          }
		          maxStringLength = currentMaxStringLength;
		          // The Classfile's access_flags field is just after the last constant pool entry.
		          /**
		           * currentCpInfoOffset：常量池数据已经全部遍历完存入cpInfoOffsets中，此时位置为:access_flags
		           */
		          header = currentCpInfoOffset;
		  
		          // Allocate the cache of ConstantDynamic values, if there is at least one.
		          constantDynamicValues = hasConstantDynamic ? new ConstantDynamic[constantPoolCount] : null;
		  
		          // Read the BootstrapMethods attribute, if any (only get the offset of each method).
		          bootstrapMethodOffsets =
		                  hasBootstrapMethods ? readBootstrapMethodsAttribute(currentMaxStringLength) : null;
		      }
		  }
		  ```
	- ### 属性分析：
	  collapsed:: true
		- int[] cpInfoOffsets：//数据的索引信息:标识了classFileBuffer中数据里包含的常量池位置
			- 由class文件往后读取8个字节，在classFile中可知（魔数u4，次版本号u2，主版本号u2）constant_pool_count即常量池大小为cpInfoOffsets数组大小，数组中数据为当前常量在classFile中的偏移量，用于快速获取常量；
		- header:
			- 存储当前类的access_flags标识位在字节码数组中位置：快速定位到当前类，父类，字段，方法等数据内容，如何验证，看一下accept()方法：
	- ### accept 函数分析
	  collapsed:: true
		- 源代码
		  collapsed:: true
			- ```java
			  // 使给定访问者访问传递给此ClassReader构造函数的JVMS ClassFile结构。
			  // 参数：
			  // classVisitor–必须访问该类的访问者。
			  // attributePrototypes–在类访问期间必须解析的属性的原型。任何类型不等于原型类型的属性都不会被解析：它的字节数组值将不变地传递给ClassWriter。如果此值包含对常量池的引用，或者与已由读取器和写入器之间的类适配器转换的类元素具有语法或语义链接，则可能会损坏它。
			  // parsingOptions–用于分析此类的选项。SKIP_CODE、SKIP_DEBUG、SKIP_FRAMES或EXPAND_FRAMES中的一个或多个。
			  public void accept(ClassVisitor classVisitor, Attribute[] attributePrototypes, int parsingOptions) {
			          ...
			          int currentOffset = this.header; //currentOffset指定位置为：u2 access_flags
			          int accessFlags = this.readUnsignedShort(currentOffset); //获取类标识位 Short
			          String thisClass = this.readClass(currentOffset + 2, charBuffer); // +2获取当前类索引 this_class
			          String superClass = this.readClass(currentOffset + 4, charBuffer); // +4 获取当前父类索引 super_class
			          String[] interfaces = new String[this.readUnsignedShort(currentOffset + 6)]; //接口集合数据：大小 + 6
			          currentOffset += 8; //u2:access_flags, u2:this_class, u2:super_class , u2:interfaces_count
			  				
			  	//获取实现接口数据
			  	int innerClassesOffset;
			          for(innerClassesOffset = 0; innerClassesOffset < interfaces.length; ++innerClassesOffset) {
			              interfaces[innerClassesOffset] = this.readClass(currentOffset, charBuffer);
			              currentOffset += 2; //interfaces[] 数组：数据类型u2
			          }
			          
			          ...
			          //获取属性表位置：attribute_info
			          int currentAttributeOffset = this.getFirstAttributeOffset();
			  	//属性表个数：attributes_count
			          int fieldsCount;
			          for(fieldsCount = this.readUnsignedShort(currentAttributeOffset - 2); fieldsCount > 0; --fieldsCount) {
			              String attributeName = this.readUTF8(currentAttributeOffset, charBuffer);
			              int attributeLength = this.readInt(currentAttributeOffset + 2);
			              currentAttributeOffset += 6;
			              ...
			              if ("Signature".equals(attributeName)) { //若当前类属性有泛型，则读取其信息
			                  signature = this.readUTF8(currentAttributeOffset, charBuffer);
			              } 
			  						...
			              currentAttributeOffset += attributeLength;
			                          
			          }
			          
			          
			      //调用visit方法，每个类只会调用一次，参数为我们读取到字节码数据
			      classVisitor.visit(this.readInt(this.cpInfoOffsets[1] - 7), accessFlags, thisClass, signature, superClass, interfaces);
			       	...
			      
			      //获取filed字段个数
			      fieldsCount = this.readUnsignedShort(currentOffset);
			      //readField() 调用classVisitor.visitField()方法
			      for(currentOffset += 2; fieldsCount-- > 0; currentOffset = this.readField(classVisitor, context, currentOffset)) {}
			      //获取method方法个数
			      methodsCount = this.readUnsignedShort(currentOffset);
			      //readMethod() 调用classVisitor.visitMethod()方法
			      for(currentOffset += 2; methodsCount-- > 0; currentOffset = this.readMethod(classVisitor, context, currentOffset)) {}
			  
			      classVisitor.visitEnd();	
			  ```
		- 作用：
			- 1、根据header快速获取类标识位，当前类索引，父类索引，接口数据，属性表等数据后调用classVisitor.visit()方法，这也就解释了为何classVisitor.visit()及classVisitor.visitEnd()只会调用一次，且一个在前，一个在最后调用；
			- 2、根据字段个数，调用readField中对字段解析调用classVisitor.visitField()方法
			- 3、根据方法个数，调用readMethod中对字段解析调用classVisitor.visitMethod()方法
	- ### readField() 和 readMethod()
	  collapsed:: true
		- 代码：
		  collapsed:: true
			- ```java
			  private int readField(ClassVisitor classVisitor, Context context, int fieldInfoOffset) {
			  	//descriptor :字段描述符 int:I ; constantValue :字段默认值
			    	FieldVisitor fieldVisitor = classVisitor.visitField(accessFlags, name, descriptor, signature,constantValue);
			    	...
			    	fieldVisitor.visitEnd();
			    	return currentOffset;
			  }
			  
			  private int readMethod(ClassVisitor classVisitor, Context context, int methodInfoOffset) {
			  	//调用classVisitor.visitMethod()扫描类中每一个方法
			    	MethodVisitor methodVisitor = classVisitor.visitMethod(context.currentMethodAccessFlags, context.currentMethodName, context.currentMethodDescriptor, signatureIndex == 0 ? null : this.readUtf(signatureIndex, charBuffer), exceptions);
			    	...
			    	//获取方法注解
			    	if (annotationDefaultOffset != 0) {
			    	  	AnnotationVisitor annotationVisitor = methodVisitor.visitAnnotationDefault();
			    	  	this.readElementValue(annotationVisitor, annotationDefaultOffset, (String)null, charBuffer);
			    	  	if (annotationVisitor != null) {
			    	  	  	annotationVisitor.visitEnd();
			    	  	}
			    	}
			    	...
			    	//如果方法存在code属性
			    	if (codeOffset != 0) {
			    	  	methodVisitor.visitCode();
			    	  	this.readCode(methodVisitor, context, codeOffset);
			    	}
			  
			    	methodVisitor.visitEnd();
			    	return currentOffset;
			  }
			  ```
		- MethodVisitor:
			- 调用顺序由classVisitor.accept() -> classVisitor.visitMethod() -> methodVisitor.visitCode() -> readCode() ->methodVisitor.visitEnd()
			- methodVisitor.visitCode()方法起始位置调用，methodVisitor.visitEnd()方法结束时调用,且只会调用一次
			- readCode() 读取code中属性值调用MethodVisitor对应方法
	- ### readCode()
		- 代码：
			- ```
			  ```
-