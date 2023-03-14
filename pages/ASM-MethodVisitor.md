- 访问Java方法的访问者。必须按以下顺序调用该类的方法
  collapsed:: true
	- ( visitParameter )
	- [ visitAnnotationDefault ]
	- ( visitAnnotation |
	- visitAnnotableParameterCount |
	- visitParameterAnnotation
	- visitTypeAnnotation |
	- visitAttribute )* [
	- visitCode (
	- visitFrame |
	- visit<i>X</i>Insn |
	- visitLabel |
	- visitInsnAnnotation |
	- visitTryCatchBlock |
	- visitTryCatchAnnotation |
	- visitLocalVariable |
	- visitLocalVariableAnnotation |
	- visitLineNumber )*
	- visitMaxs ]
	- visitEnd.
- ## 1、visitParameter()
	- 介绍：它用于访问方法的参数信息。
	- 使用：
	  collapsed:: true
		- 在 Java 中，方法参数是指在方法声明中列出的参数列表。在 ASM 中，通过 visitParameter 方法可以访问方法的参数信息，包括参数名称和访问修饰符。需要注意的是，该方法只能在 visitCode 方法之前调用。
		- visitParameter 方法会在访问方法时被调用，用于访问方法的参数信息。其中，name 参数表示参数的名称，access 参数表示参数的访问修饰符。在访问完所有参数信息后，需要调用 visitEnd 方法，通知 MethodVisitor 方法的访问结束了。
		- 需要注意的是，由于 JDK 1.8 之前的版本中，Java 字节码并没有将方法的参数名称保存在 class 文件中，因此 name 参数可能为 null。在 JDK 1.8 及以后的版本中，Java 字节码开始支持保存方法参数的名称信息。如果编译时使用了 -parameters 参数，则可以在 class 文件中保存方法参数的名称。
	- code:
	  collapsed:: true
		- ```java
		  name：表示参数的名称，以字符串形式给出。
		  access：表示参数的访问修饰符，以整数形式给出。
		  void visitParameter(String name, int access);
		  ```
- ## 2、visitAnnotationDefault()
  collapsed:: true
	- 介绍：用于访问注解的默认值。
	- 使用：
	  collapsed:: true
		- 在 Java 中，注解可以包含元素，元素可以有默认值。如果在注解使用时没有指定元素的值，则会使用默认值。在 ASM 中，可以通过 visitAnnotationDefault 方法来访问注解的默认值。该方法返回一个 AnnotationVisitor，用于访问注解的默认值。
		- 需要注意的是，该方法只能在 visitAnnotation 方法之后和 visitCode 方法之前调用。
		- visitAnnotationDefault 方法返回一个 AnnotationVisitor 对象，用于访问注解的默认值。访问注解默认值的方式与访问其他注解信息的方式类似，需要使用 AnnotationVisitor 中的方法来访问注解的元素信息。
		- 例如，如果注解的默认值为 @MyAnnotation("default")，则可以通过以下代码来访问注解的默认值：
			- ```java
			  AnnotationVisitor av = mv.visitAnnotationDefault();
			  av.visit("value", "default");
			  av.visitEnd();
			  
			  ```
		- 在访问完注解的默认值后，需要调用 visitEnd 方法，通知 MethodVisitor 默认值的访问结束了。
	- code:
- ## 3、visitAnnotation()
  collapsed:: true
	- 介绍：用于访问方法上的注解信息。
	- 使用：
	  collapsed:: true
		- 在 Java 中，注解可以用于方法上，用于给方法添加一些元数据信息。在 ASM 中，可以通过 visitAnnotation 方法来访问方法上的注解信息。
		- visitAnnotation 方法会在访问方法时被调用，用于访问方法上的注解信息。其中，descriptor 参数表示注解的类型描述符，visible 参数表示注解是否可见。在访问注解信息时，需要返回一个 AnnotationVisitor 对象，用于访问注解的元素信息。
		- 需要注意的是，如果注解不可见，则在使用反射 API 获取注解信息时可能无法获取到该注解。如果需要在运行时获取注解信息，建议将注解设置为可见。
		- 例如，如果方法上有 @MyAnnotation("value") 注解，则可以通过以下代码来访问该注解的信息：
		- ```java
		  AnnotationVisitor av = mv.visitAnnotation("Lcom/example/MyAnnotation;", true);
		  av.visit("value", "value");
		  av.visitEnd();
		  
		  ```
		- 在访问完注解信息后，需要调用 visitEnd 方法，通知 MethodVisitor 注解信息的访问结束了。
	- code:
	  collapsed:: true
		- ```java
		  descriptor：表示注解的类型描述符，以字符串形式给出。
		  例如，对于 @MyAnnotation 注解，其类型描述符为 "Lcom/example/MyAnnotation;"。
		  visible：表示注解是否可见，以布尔值形式给出。
		  AnnotationVisitor visitAnnotation(String descriptor, boolean visible);
		  ```
- ## 4、visitTypeAnnotation()
  collapsed:: true
	- 介绍：用于访问方法上的类型注解信息。
	- 使用：
		- 在 Java 中，类型注解用于指定程序中的类型使用情况。在 ASM 中，可以通过 visitTypeAnnotation 方法来访问方法上的类型注解信息。
		- visitTypeAnnotation 方法会在访问方法时被调用，用于访问方法上的类型注解信息。其中，typeRef 参数表示类型引用，typePath 参数表示类型路径，descriptor 参数表示注解的类型描述符，visible 参数表示注解是否可见。在访问类型注解信息时，需要返回一个 AnnotationVisitor 对象，用于访问注解的元素信息。
		- 例如，如果方法上有 @MyAnnotation("value") 类型注解，则可以通过以下代码来访问该类型注解的信息：
		- ```java
		  AnnotationVisitor av = mv.visitTypeAnnotation(TypeReference.METHOD_TYPE_PARAMETER, new TypePath("L"), "Lcom/example/MyAnnotation;", true);
		  av.visit("value", "value");
		  av.visitEnd();
		  
		  ```
		- 在访问完类型注解信息后，需要调用 visitEnd 方法，通知 MethodVisitor 类型注解信息的访问结束了。
	- code:
		- ```java
		  
		  typeRef：表示类型引用，以整数形式给出。
		  typePath：表示类型路径，以 TypePath 对象形式给出。
		  descriptor：表示注解的类型描述符，以字符串形式给出。例如，对于 @MyAnnotation 注解，其类型描述符为 "Lcom/example/MyAnnotation;"。
		  visible：表示注解是否可见，以布尔值形式给出。
		  AnnotationVisitor visitTypeAnnotation(int typeRef, TypePath typePath, String descriptor, boolean visible);
		  
		  ```
- ## 5、visitAnnotableParameterCount()
  collapsed:: true
	- 介绍：用于通知 ASM 框架方法参数中存在注解的参数数量。
	- 使用：
	  collapsed:: true
		- 在 Java 中，可以为方法参数添加注解。在 ASM 中，可以通过 visitAnnotableParameterCount 方法来通知 ASM 框架方法参数中存在注解的参数数量。当访问到此方法时，需要在访问参数之前调用 visitParameterAnnotation 方法来访问每个参数的注解信息。
		- 例如，如果方法有一个参数 foo，并且有 @MyAnnotation 注解，则可以使用以下代码来访问该注解信息：
		- ```java
		  mv.visitAnnotableParameterCount(1, true);
		  mv.visitParameterAnnotation(0, "Lcom/example/MyAnnotation;", true);
		  
		  ```
		- 在上面的代码中，visitAnnotableParameterCount 方法被调用，传入参数 1 表示存在一个带注解的参数，true 表示注解可见。然后，调用 visitParameterAnnotation 方法来访问参数的注解信息。其中，0 表示参数的索引，"Lcom/example/MyAnnotation;" 表示注解的类型描述符，true 表示注解可见。
		- 需要注意的是，调用 visitParameterAnnotation 方法时，必须先调用 visitAnnotableParameterCount 方法。否则，ASM 框架将无法识别方法参数中存在注解的参数数量。
	- code:
		- ```java
		  parameterCount：表示方法参数中存在注解的参数数量。
		  visible：表示注解是否可见，以布尔值形式给出。
		  void visitAnnotableParameterCount(int parameterCount, boolean visible);
		  
		  ```
- ## 6、visitParameterAnnotation()
  collapsed:: true
	- 介绍：用于访问方法参数的注解信息。
	- 使用：
		- 在 Java 中，可以为方法参数添加注解。在 ASM 中，可以使用 visitParameterAnnotation 方法来访问方法参数的注解信息。当访问到此方法时，需要在访问参数之前调用 visitAnnotableParameterCount 方法来通知 ASM 框架方法参数中存在注解的参数数量。
		- 例如，如果方法有一个参数 foo，并且有 @MyAnnotation 注解，则可以使用以下代码来访问该注解信息：
		- ```java
		  mv.visitAnnotableParameterCount(1, true);
		  mv.visitParameterAnnotation(0, "Lcom/example/MyAnnotation;", true);
		  
		  ```
		- 在上面的代码中，visitAnnotableParameterCount 方法被调用，传入参数 1 表示存在一个带注解的参数，true 表示注解可见。然后，调用 visitParameterAnnotation 方法来访问参数的注解信息。其中，0 表示参数的索引，"Lcom/example/MyAnnotation;" 表示注解的类型描述符，true 表示注解可见。
		- 在调用 visitParameterAnnotation 方法后，将返回一个 AnnotationVisitor 对象，可以使用该对象来访问注解的元素值。例如，可以使用以下代码访问 @MyAnnotation 注解中 value 元素的值：
		- ```java
		  AnnotationVisitor av = mv.visitParameterAnnotation(0, "Lcom/example/MyAnnotation;", true);
		  av.visit("value", "hello");
		  av.visitEnd();
		  
		  ```
		- 在上面的代码中，visit 方法被调用，传入参数 "value" 表示要访问注解中的 value 元素，"hello" 表示该元素的值。然后，调用 visitEnd 方法来结束对注解的访问。
		- 需要注意的是，调用 visitParameterAnnotation 方法时，必须先调用 visitAnnotableParameterCount 方法。否则，ASM 框架将无法识别方法参数中存在注解的参数数量。
	- code:
		- ```java
		  parameter：表示要访问注解的参数索引。
		  descriptor：表示注解的类型描述符。
		  visible：表示注解是否可见，以布尔值形式给出。
		  
		  AnnotationVisitor visitParameterAnnotation(int parameter, String descriptor, boolean visible);
		  
		  ```
- ## 7、visitAttribute()
  collapsed:: true
	- 介绍：用于访问类、字段或方法的属性信息。
	- 使用：
		- 在 ASM 中，可以向类、字段或方法添加自定义属性，这些属性是由用户定义的，并且不在 Java 类或接口规范中定义。当访问类、字段或方法时，可以使用 visitAttribute 方法来访问这些自定义属性。
		- 属性对象由 Attribute 类或其子类的实例表示，其中 Attribute 是一个抽象类，定义了属性对象的基本行为。在 ASM 中，可以继承 Attribute 类并实现自定义的属性对象。例如，以下代码定义了一个名为 MyAttribute 的自定义属性对象：
		- ```java
		  public class MyAttribute extends Attribute {
		      public static final String NAME = "MyAttribute";
		  
		      public MyAttribute() {
		          super(NAME);
		      }
		  
		      // Override other methods as needed
		  }
		  
		  ```
		- 要向类、字段或方法添加自定义属性，可以使用 visitAttribute 方法。例如，以下代码将自定义属性 MyAttribute 添加到类中
		- ```java
		  ClassWriter cw = new ClassWriter(ClassWriter.COMPUTE_MAXS);
		  cw.visit(Opcodes.V1_8, Opcodes.ACC_PUBLIC, "com/example/MyClass", null, "java/lang/Object", null);
		  Attribute attr = new MyAttribute();
		  cw.visitAttribute(attr);
		  
		  ```
		- 在上面的代码中，首先创建了一个 ClassWriter 对象，并使用 visit 方法访问类 com.example.MyClass。然后，创建了一个自定义属性 MyAttribute 的实例，并使用 visitAttribute 方法将其添加到类中。
		- 当访问类、字段或方法时，可以使用 getAttribute 方法获取已添加的属性信息。例如，以下代码获取类中名为 MyAttribute 的属性对象：
		- ```java
		  Attribute attr = cw.getAttribute(MyAttribute.NAME);
		  if (attr != null) {
		      // Process the attribute
		  }
		  
		  ```
		- 在上面的代码中，getAttribute 方法被调用，传入参数 MyAttribute.NAME 表示要获取名为 MyAttribute 的属性对象。如果该属性存在，则将返回一个 Attribute 对象，可以对其进行进一步处理。
	- code:
		- ```java
		  attribute：表示要访问的属性对象。
		  void visitAttribute(Attribute attribute);
		  
		  ```
- ## 8、visitCode()
  collapsed:: true
	- 介绍: 接口的一个方法，用于访问方法的字节码指令。该方法必须在访问方法的其他部分之后调用。
	- 使用:
		- 该方法没有参数，因为在访问方法字节码之前，所有需要设置的信息（例如局部变量表和异常表）都应该已经设置好了。
		- 在 visitCode 方法之后，可以使用 visitInsn、visitIntInsn、visitVarInsn、visitTypeInsn、visitFieldInsn、visitMethodInsn、visitJumpInsn、visitLabel、visitLdcInsn、visitIincInsn 和 visitTableSwitchInsn 等方法访问方法的字节码指令。
		- 例如，以下代码访问一个名为 foo 的方法的字节码：
		- ```java
		  MethodVisitor mv = ...; // create MethodVisitor
		  mv.visitCode();
		  mv.visitInsn(Opcodes.ICONST_0);
		  mv.visitVarInsn(Opcodes.ISTORE, 1);
		  Label loop = new Label();
		  mv.visitLabel(loop);
		  mv.visitIincInsn(1, 1);
		  mv.visitVarInsn(Opcodes.ILOAD, 1);
		  mv.visitIntInsn(Opcodes.BIPUSH, 10);
		  mv.visitJumpInsn(Opcodes.IF_ICMPLT, loop);
		  mv.visitInsn(Opcodes.RETURN);
		  mv.visitMaxs(2, 2);
		  mv.visitEnd();
		  
		  ```
		- 在上面的代码中，首先创建了一个 MethodVisitor 对象，然后使用 visitCode 方法开始访问方法字节码。接下来，使用 visitInsn 方法添加一条 ICONST_0 指令，该指令将整数值 0 推入操作数栈。然后，使用 visitVarInsn 方法添加一条 ISTORE_1 指令，该指令将栈顶的整数值存储到局部变量 1 中。接下来，使用 visitLabel 方法添加一个标签，表示一个循环的开始。然后，使用 visitIincInsn 方法添加一条 IINC 指令，该指令将局部变量 1 的值增加 1。接下来，使用 visitVarInsn 方法添加一条 ILOAD_1 指令，该指令将局部变量 1 的值推入操作数栈。然后，使用 visitIntInsn 方法添加一条 BIPUSH 指令，该指令将一个 byte 类型的整数值推入操作数栈。接下来，使用 visitJumpInsn 方法添加一条 IF_ICMPGE 指令，该指令比较两个整数值，如果值在栈中的前者小于或等于后者，则跳转到指定的标签处。在上面的代码中，跳转指令将跳转到循环的开始处。最后，使用 visitInsn 方法添加一条 RETURN 指令，表示方法的结束。在所有字节码指令添加完毕后，使用 visitMaxs 方法指定操作数
	- code:
- ## 9、visitFrame()
  collapsed:: true
	- 介绍：用于访问Java字节码中方法的栈帧信息。
	- 使用：
	- code:
		- 参数：
		  collapsed:: true
			- type：表示栈帧的类型，它是一个整型值，可以取下列常量之一：
				- Opcodes.F_NEW：表示一个新的栈帧，此时参数numLocal表示局部变量表的大小，参数local表示局部变量表，参数numStack表示操作数栈的大小，参数stack表示操作数栈。
				- Opcodes.F_FULL：表示一个完整的栈帧，此时参数numLocal表示局部变量表的大小，参数local表示局部变量表，参数numStack表示操作数栈的大小，参数stack表示操作数栈。
				- Opcodes.F_APPEND：表示一个栈帧追加，此时参数numLocal表示追加的局部变量表项数目，参数local表示追加的局部变量表。
				- Opcodes.F_CHOP：表示一个栈帧截断，此时参数numLocal表示截断的局部变量表项数目。
				- Opcodes.F_SAME：表示一个空栈帧。
				- Opcodes.F_SAME1：表示一个栈帧，其中一个操作数栈项的值与上一个栈帧相同。
			- numLocal：表示该栈帧的局部变量表的大小，它是一个整型值。
			- local：表示该栈帧的局部变量表，它是一个Object类型的数组，数组长度为numLocal，每个数组元素表示一个局部变量。在栈帧类型为Opcodes.F_APPEND或Opcodes.F_FULL时，local参数包含了整个局部变量表。在栈帧类型为Opcodes.F_NEW时，local参数表示了局部变量表中的前numLocal项。
			- numStack：表示该栈帧的操作数栈的大小，它是一个整型值。
			- stack：
				- 此帧中的操作数堆栈类型。不得修改此数组。其内容具有与“本地”数组相同的格式。
		- ```java
		  
		  public void visitFrame(
		        final int type,
		        final int numLocal,
		        final Object[] local,
		        final int numStack,
		        final Object[] stack)
		  ```
- ## 10、visitInsn()
	- 介绍：它用于访问方法中的单条指令。
	- 使用：
	- code:
		-
- ## 11、visitIntInsn()
- ## 12、visitVarInsn()
- ## 13、visitTypeInsn()
- ## 14、visitFieldInsn()
- ## 15、visitMethodInsn()
- ## 16、visitMethodInsn()
- ## 17、visitInvokeDynamicInsn()
- ## 18、visitJumpInsn()
- ## 19、visitLabel()
- ## 20、visitLdcInsn()
- ## 21、visitIincInsn()
- ## 22、visitTableSwitchInsn()
- ## 23、visitLookupSwitchInsn()
- ## 24、visitMultiANewArrayInsn()
- ## 25、visitInsnAnnotation()
- ## 26、visitTryCatchBlock()
- ## 27、visitTryCatchAnnotation()
- ## 28、visitLocalVariable()
- ## 29、visitLocalVariableAnnotation()
- ## 30、visitLineNumber()
- ## 31、visitMaxs()
- ## 32、visitEnd()
-
-