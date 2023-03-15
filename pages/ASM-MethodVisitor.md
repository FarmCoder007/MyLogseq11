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
  collapsed:: true
	- 介绍：它用于访问方法中的单条指令。
	- 使用：
		- visitInsn方法通常用于处理那些不需要任何操作数的指令，例如将两个数字相加的iadd指令，或者创建一个新对象的new指令。在方法访问器访问到这些指令时，会调用visitInsn方法，并传递指令的操作码作为参数。
		- 在方法访问器中，如果需要对某个指令进行特殊的处理，可以重载visitInsn方法，根据指令的操作码来判断是否需要特殊处理，如果需要，则在方法中进行相应的处理逻辑。
	- code:
		- ```java
		  opcode：表示指令的操作码。它是一个整数，取值在Opcodes类中定义。
		  public void visitInsn(final int opcode) {
		  ```
- ## 11、visitIntInsn()
  collapsed:: true
	- 介绍：它用于访问方法中的整数指令。
	- 使用：
		- visitIntInsn方法通常用于处理那些包含整数操作数的指令，例如将一个常量加载到栈上的iconst指令，或者将一个整数类型的本地变量加载到栈上的iload指令。在方法访问器访问到这些指令时，会调用visitIntInsn方法，并传递指令的操作码和操作数作为参数。
		- 在方法访问器中，如果需要对某个整数指令进行特殊的处理，可以重载visitIntInsn方法，根据指令的操作码来判断是否需要特殊处理，如果需要，则在方法中进行相应的处理逻辑。
	- code:
		- ```java
		  opcode：表示指令的操作码。它是一个整数，取值在Opcodes类中定义。
		  operand：表示指令中包含的整数操作数。它是一个int类型的值，表示指令的操作数。
		  public void visitIntInsn(final int opcode, final int operand) {
		  ```
- ## 12、visitVarInsn()
  collapsed:: true
	- 介绍：它用于访问方法中的本地变量指令。
	- 使用：
		- visitVarInsn方法通常用于处理那些访问或修改本地变量的指令，例如将一个整数类型的本地变量加载到栈上的iload指令，或者将一个对象类型的本地变量存储到栈上的astore指令。在方法访问器访问到这些指令时，会调用visitVarInsn方法，并传递指令的操作码和本地变量的编号作为参数。
		- 在方法访问器中，如果需要对某个本地变量指令进行特殊的处理，可以重载visitVarInsn方法，根据指令的操作码来判断是否需要特殊处理，如果需要，则在方法中进行相应的处理逻辑。
	- code:
		- ```java
		  opcode：表示指令的操作码。它是一个整数，取值在Opcodes类中定义。
		  var：表示指令中操作的本地变量的编号。它是一个整数，
		  从0开始计数。在Java字节码中，本地变量可以是基本数据类型或者对象类型。
		  public void visitVarInsn(final int opcode, final int var) {
		  ```
- ## 13、visitTypeInsn()
  collapsed:: true
	- 介绍：它用于访问方法中的类型指令。
	- 使用：
		- visitTypeInsn方法通常用于创建新对象、访问静态字段、检查实例类型等操作。例如，创建一个新对象的指令是new，它需要指定要创建的对象的类型，就可以使用visitTypeInsn方法传递new操作码和要创建的对象的类型作为参数。
		- 在方法访问器中，如果需要对某个类型指令进行特殊的处理，可以重载visitTypeInsn方法，根据指令的操作码来判断是否需要特殊处理，如果需要，则在方法中进行相应的处理逻辑。
	- code:
		- ```java
		  opcode：表示指令的操作码。它是一个整数，取值在Opcodes类中定义。
		  type：表示被操作的类型。它是一个字符串，表示要创建或操作的类的类型。在Java字节码中，类型指令用于操作类、接口、数组等类型。
		  
		  public void visitTypeInsn(final int opcode, final String type) {
		  ```
- ## 14、visitFieldInsn()
	- 介绍：visitFieldInsn方法通常用于访问类的字段
	- 使用：
		- visitFieldInsn方法通常用于访问类的字段，包括访问静态字段和访问实例字段。例如，访问一个类的静态字段的指令是GETSTATIC，需要指定要访问的类、字段名称和字段类型，就可以使用visitFieldInsn方法传递GETSTATIC操作码、要访问的类的类型、要访问的静态字段的名称和类型描述符作为参数。
		- 在方法访问器中，如果需要对某个字段指令进行特殊的处理，可以重载visitFieldInsn方法，根据指令的操作码和要访问的字段的类型来判断是否需要特殊处理，如果需要，则在方法中进行相应的处理逻辑。
	- code:
		- ```java
		  opcode：表示指令的操作码。它是一个整数，取值在Opcodes类中定义。
		  要访问的类型指令的操作码。此操作码是GETSTATIC 获取类的静态字段
		                              PUTSTATIC
		                                 GETFIELD。
		  owner：表示要访问字段的类的类型。它是一个字符串，表示包含要访问字段的类的类型。
		  name：表示要访问的字段的名称。它是一个字符串，表示要访问的字段的名称。
		  descriptor：表示要访问字段的类型。它是一个字符串，表示要访问的字段的类型描述符。
		  
		  public void visitFieldInsn(
		        final int opcode, final String owner, final String name, final String descriptor) {
		  ```
- ## 15、visitMethodInsn() 废弃
	- 介绍：用于访问方法中的方法指令。
	- 使用：
		- visitMethodInsn方法通常用于访问类的方法，包括访问静态方法和访问实例方法。例如，访问一个类的静态方法的指令是invokestatic，需要指定要访问的类、方法名称和方法类型，就可以使用visitMethodInsn方法传递invokestatic操作码、要访问的类的类型、要访问的静态方法的名称和类型描述符作为参数。
		- 在方法访问器中，如果需要对某个方法指令进行特殊的处理，可以重载visitMethodInsn方法，根据指令的操作码和要访问的方法的类型来判断是否需要特殊处理，如果需要，则在方法中进行相应的处理逻辑。
	- code
		- 参数详解：
			- descriptor：
		- ```java
		  opcode：表示指令的操作码。它是一个整数，取值在Opcodes类中定义。
		  owner：表示要访问方法的类的类型。它是一个字符串，表示包含要访问方法的类的类型。
		  name：表示要访问的方法的名称。它是一个字符串，表示要访问的方法的名称。
		  descriptor：表示要访问方法的描述符。它是一个字符串，表示要访问的方法的类型描述符。
		  @Deprecated
		    public void visitMethodInsn(
		        final int opcode, final String owner, final String name, final String descriptor) {
		  ```
- ## 16、visitMethodInsn() 新
	- 介绍：通常用于访问类的方法
	- 使用：
		- visitMethodInsn方法通常用于访问类的方法，包括访问静态方法和访问实例方法。例如，访问一个类的静态方法的指令是invokestatic，需要指定要访问的类、方法名称和方法类型，就可以使用visitMethodInsn方法传递invokestatic操作码、要访问的类的类型、要访问的静态方法的名称和类型描述符作为参数。
		- 在方法访问器中，如果需要对某个方法指令进行特殊的处理，可以重载visitMethodInsn方法，根据指令的操作码和要访问的方法的类型来判断是否需要特殊处理，如果需要，则在方法中进行相应的处理逻辑。
	- code:
		- ```java
		  opcode：方法调用指令的操作码，例如 INVOKEVIRTUAL、INVOKESTATIC 等。该参数的值应该通过 Opcodes 类中的常量进行设置。
		  owner：被调用方法的所有者类名，即该方法所在类的类名。例如，调用 java.lang.String 类中的 length() 方法，则该参数应该设置为 java/lang/String。
		  name：被调用的方法名。例如，调用 java.lang.String 类中的 length() 方法，则该参数应该设置为 length。
		  descriptor：被调用方法的描述符。描述符用来描述方法的参数类型和返回值类型，例如 (Ljava/lang/String;)I 表示参数为 java.lang.String，返回值为 int 类型。
		  isInterface：指示被调用方法是否为接口方法。如果被调用方法是接口方法，则该参数应该设置为 true；否则，应该设置为 false。
		  public void visitMethodInsn(
		        final int opcode,
		        final String owner,
		        final String name,
		        final String descriptor,
		        final boolean isInterface) {
		  ```
- ## 17、visitInvokeDynamicInsn()
	- 介绍：用于访问Java 7中引入的InvokeDynamic指令。
	- 使用：
		- Java 7中的InvokeDynamic指令是为了支持动态语言在JVM上的执行而引入的。它允许开发人员在运行时动态地创建方法调用点，从而使得动态语言的执行更加高效。
		- 在方法访问器中，如果需要对InvokeDynamic指令进行特殊的处理，可以重载visitInvokeDynamicInsn方法，在方法中进行相应的处理逻辑。例如，可以利用该方法在方法访问器中实现动态代理的功能。
	- code:
		- ```java
		  name：表示要访问的方法的名称。它是一个字符串，表示要访问的方法的名称。
		  descriptor：表示要访问方法的描述符。它是一个字符串，表示要访问的方法的类型描述符。
		  bootstrapMethodHandle：表示引导方法的句柄。它是一个Handle类型的对象，表示引导方法的类型、名称和描述符。
		  bootstrapMethodArguments：表示引导方法的参数。
		  它是一个对象数组，包含了引导方法需要的参数。引导方法的参数可以是基本类型、字符串、类型或者句柄。
		  public void visitInvokeDynamicInsn(
		        final String name,
		        final String descriptor,
		        final Handle bootstrapMethodHandle,
		        final Object... bootstrapMethodArguments) {
		  ```
- ## 18、visitJumpInsn()
	- 介绍：用于访问Java字节码中的跳转指令。
	- 使用：
		- 在Java字节码中，跳转指令用于控制程序的执行流程。通过调用visitJumpInsn方法，我们可以访问并修改Java字节码中的跳转指令。在ASM框架中，我们可以通过实现MethodVisitor接口，重载visitJumpInsn方法来进行跳转指令的处理。
		- 需要注意的是，在修改跳转指令时，需要保证修改后的指令仍然符合Java字节码的规范，否则会导致程序执行错误。因此，在修改跳转指令时，需要仔细阅读Java字节码规范，并进行必要的验证和测试。
	- code:
		- ```java
		  opcode：表示跳转指令的操作码。它是一个整数，表示跳转指令的类型。
		  label：表示跳转指令的目标标签。它是一个Label类型的对象，表示跳转指令的目标地址。
		  public void visitJumpInsn(final int opcode, final Label label) {
		  ```
- ## 19、visitLabel()
	- 介绍：用于访问Java字节码中的标签
	- 使用：
		- 在Java字节码中，标签用于标识代码的执行位置，通常用于标识跳转指令的目标地址。通过调用visitLabel方法，我们可以访问并修改Java字节码中的标签。
		- 在ASM框架中，我们可以通过实现MethodVisitor接口，重载visitLabel方法来进行标签的处理。需要注意的是，标签是Java字节码中的重要元素，它们的位置必须精确无误，否则会导致程序执行错误。在处理标签时，我们需要仔细阅读Java字节码规范，并进行必要的验证和测试。
	- code:
		- ```java
		  label：表示需要访问的标签。它是一个Label类型的对象，表示Java字节码中的标签。
		  
		  public void visitLabel(final Label label) {
		  ```
- ## 20、visitLdcInsn()
	- 介绍：
		- 用于访问Java字节码中的LDC指令。用于访问将常量推送至栈顶的指令，如 LDC 指令。当 ASM 通过字节码分析器读取字节码指令时，如果遇到将常量推送至栈顶的指令，就会调用 visitLdcInsn 方法。
		- 并不是方法的入参
	- 使用：
		- 在Java字节码中，LDC指令用于将常量值加载到操作数栈中。通过调用visitLdcInsn方法，我们可以访问并修改Java字节码中的LDC指令。通常，我们可以通过调用ASM中提供的visitLdcInsn方法，将常量值加载到操作数栈中。
		- 需要注意的是，在Java字节码中，常量池是一个重要的概念。在处理LDC指令时，我们需要考虑常量池中的信息，并进行必要的验证和测试。同时，由于Java字节码中支持多种类型的常量，我们需要根据实际情况选择正确的visitLdcInsn方法，并传递正确的参数。
	- code:
		- ```java
		  cst：表示需要访问的常量值。它可以是数字、字符串、类类型等类型的常量。
		  public void visitLdcInsn(final Object value) {
		  ```
- ## 21、visitIincInsn()
	- 介绍：用于访问Java字节码中的IINC指令。
	- 使用：
		- 在Java字节码中，IINC指令用于对指定的局部变量进行增量操作。通过调用visitIincInsn方法，我们可以访问并修改Java字节码中的IINC指令。通常，我们可以通过调用ASM中提供的visitIincInsn方法，来对指定的局部变量进行增量操作。
		- 需要注意的是，在Java字节码中，局部变量是一个重要的概念。在处理IINC指令时，我们需要考虑局部变量表中的信息，并进行必要的验证和测试。同时，由于IINC指令支持增量操作，我们需要根据实际情况选择正确的visitIincInsn方法，并传递正确的参数。
	- code:
		- ```java
		  var：表示需要增加的局部变量的索引。
		  increment：表示需要增加的值。
		  public void visitIincInsn(final int var, final int increment) {
		  ```
- ## 22、visitTableSwitchInsn()
	- 介绍：用于访问Java字节码中的TABLESWITCH指令。
	- 使用：
		- 在Java字节码中，TABLESWITCH指令用于实现一个带有多个分支的switch语句。通过调用visitTableSwitchInsn方法，我们可以访问并修改Java字节码中的TABLESWITCH指令。通常，我们可以通过调用ASM中提供的visitTableSwitchInsn方法，来生成一个新的TABLESWITCH指令。
		- 需要注意的是，在Java字节码中，TABLESWITCH指令的操作比较复杂。在处理TABLESWITCH指令时，我们需要考虑各个分支的跳转目标，并进行必要的验证和测试。同时，由于TABLESWITCH指令支持多个分支，我们需要根据实际情况选择正确的visitTableSwitchInsn方法，并传递正确的参数。
	- code:
		- ```java
		  min：表示case语句的最小值。
		  max：表示case语句的最大值。
		  dflt：表示默认的跳转目标。
		  labels：表示各个case语句的跳转目标
		  public void visitTableSwitchInsn(
		        final int min, final int max, final Label dflt, final Label... labels) 
		  ```
- ## 23、visitLookupSwitchInsn()
	- 介绍：用于访问字节码中的 LOOKUPSWITCH 指令。
	- 使用：
		- LOOKUPSWITCH 指令是用于在具有多个分支的情况下进行跳转的指令。它使用给定的键将其与一个跳转目标列表进行比较，并跳转到相应的目标。如果没有匹配项，则跳转到默认目标。
	- code:
		- ```java
		  dflt：指向默认情况下应跳转到的 Label。
		  keys：一个整数数组，它包含要比较的键的列表。
		  labels：一个 Label 数组，其中包含与每个键关联的目标 Label。
		  public void visitLookupSwitchInsn(final Label dflt, final int[] keys, final Label[] labels) {
		  ```
- ## 24、visitMultiANewArrayInsn()
	- 介绍：用于访问字节码中的 MULTIANEWARRAY 指令。
	- 使用：
		- MULTIANEWARRAY 指令用于创建多维数组。它接受一个类型描述符和一个整数，该整数指定要创建的数组的维数。例如，如果 numDimensions 参数为 2，则将创建一个二维数组。指令的结果是将新创建的数组的引用压入操作数栈顶。
	- code:
		- ```java
		  descriptor：要创建的数组的类型描述符，如 [[Ljava/lang/Object;。
		  numDimensions：要创建的数组的维数。
		  public void visitMultiANewArrayInsn(final String descriptor, final int numDimensions) {
		  ```
- ## 25、visitInsnAnnotation()
	- 介绍：用于访问字节码中指令的注解。
	- 使用：
		- 此方法允许访问指令上的注解，例如 @Deprecated、@Override 等。它返回一个 AnnotationVisitor 对象，用于访问注解中的成员。AnnotationVisitor 对象将访问注解的每个成员，并通过其相应的 visit 方法返回该成员的值。
	- code:
		- ```java
		  typeRef：表示要注解的类型的引用类型。例如，可以使用 TypeReference.METHOD_RETURN 注解方法的返回值。
		  typePath：表示要注解的类型的路径。这是一个可选参数，可以为 null。
		  descriptor：注解的描述符。它指定了注解的类型，如 Ljava/lang/Deprecated;。
		  visible：表示注解是否在运行时可见。
		  
		  public AnnotationVisitor visitInsnAnnotation(
		        final int typeRef, final TypePath typePath, final String descriptor, final boolean visible) {
		  ```
- ## 26、visitTryCatchBlock()
	- 介绍：用于访问方法中的try-catch块。
	- 使用：
		- 例如，以下示例代码展示了如何使用visitTryCatchBlock方法：
		- ```java
		  public class TryCatchBlockMethodVisitor extends MethodVisitor {
		  
		      public TryCatchBlockMethodVisitor(MethodVisitor methodVisitor) {
		          super(Opcodes.ASM5, methodVisitor);
		      }
		  
		      @Override
		      public void visitTryCatchBlock(Label start, Label end, Label handler, String type) {
		          // 输出try-catch块的信息
		          System.out.println("Try-Catch Block:");
		          System.out.println("Start: " + start);
		          System.out.println("End: " + end);
		          System.out.println("Handler: " + handler);
		          System.out.println("Type: " + type);
		          // 调用父类方法，继续访问其他指令
		          super.visitTryCatchBlock(start, end, handler, type);
		      }
		  }
		  
		  ```
		- 在访问方法时，visitTryCatchBlock方法将被调用，并输出try-catch块的信息。最后，调用父类的visitTryCatchBlock方法，以便继续访问其他指令。
	- code:
		- ```java
		  start：表示try块的起始位置，即try块的第一条指令的标签。
		  end：表示try块的结束位置，即try块的最后一条指令的下一条指令的标签。
		  handler：表示异常处理器代码块的起始位置，即捕获异常后将跳转到的位置的标签。
		  type：表示捕获的异常类型的内部名称，如果不是捕获异常，则为null。
		   public void visitTryCatchBlock(
		        final Label start, final Label end, final Label handler, final String type) {
		  ```
- ## 27、visitTryCatchAnnotation()
	- 介绍：
		- 方法是在访问方法的异常处理表中的某个异常的注解时调用的。它将创建一个 AnnotationVisitor 对象，用于访问这个注解的内容。
	- 使用：
	- code:
		- ```java
		  typeRef：表示此注释类型的引用类型。引用类型是一个以 I 表示类型引用的 8 位无符号整数，例如，类型引用可以是方法的参数、方法的返回值、泛型类型参数等。
		  typePath：表示此注释类型的类型路径。类型路径是一系列用于指定注释类型在 class 文件中出现的嵌套类型或数组类型中的位置的项。
		  descriptor：表示此注释类型的描述符。
		  visible：表示此注释类型是否在运行时可见。如果为 true，则注释类型在运行时可见；否则，只有在反射时才能访问。
		  public AnnotationVisitor visitTryCatchAnnotation(
		        final int typeRef, final TypePath typePath, final String descriptor, final boolean visible) {
		  ```
- ## 28、visitLocalVariable()
	- 介绍：用于访问方法的本地变量
	- 使用：
		- 方法访问者（MethodVisitor）将在访问方法体时调用此方法。此方法的调用顺序必须与本地变量在字节码中的顺序一致。通过 visitCode 方法进入方法体，通过 visitMaxs 方法离开方法体。
	- code:
		- ```java
		  name：本地变量的名称
		  descriptor：本地变量的类型描述符
		  signature：本地变量的类型签名（可选）
		  start：本地变量的起始偏移量
		  end：本地变量的结束偏移量
		  index：本地变量的索引
		  public void visitLocalVariable(
		        final String name,
		        final String descriptor,
		        final String signature,
		        final Label start,
		        final Label end,
		        final int index) {
		  ```
- ## 29、visitLocalVariableAnnotation()
	- 介绍：用于访问局部变量的注解。
	- 使用：
		- 该方法返回一个 AnnotationVisitor 对象，用于访问局部变量的注解。这个 AnnotationVisitor 的方法可以用来访问该注解的信息（名称和值）。
	- code:
		- ```java
		  
		  typeRef：一个目标类型说明符，指示了这个局部变量注解是如何与方法调用相关联的。
		  typePath：表示类型注解中的路径。
		  start：表示局部变量在字节码中起始的范围（使用标签数组表示）。
		  end：表示局部变量在字节码中结束的范围（使用标签数组表示）。
		  index：表示局部变量在方法中的索引。
		  descriptor：注解描述符。
		  visible：表示注解是否在运行时可见。
		  public AnnotationVisitor visitLocalVariableAnnotation(
		        final int typeRef,
		        final TypePath typePath,
		        final Label[] start,
		        final Label[] end,
		        final int[] index,
		        final String descriptor,
		        final boolean visible) {
		  ```
- ## 30、visitLineNumber()
	- 介绍：
		- 接口中的一个方法，用于访问Java源代码中的行号信息。它的作用是为当前正在访问的方法添加一个源代码行号和与之对应的指令索引
	- 使用：
		- 例如，下面的示例代码使用visitLineNumber在生成的字节码中添加了行号信息：
		- ```java
		  @Override
		  public void visitCode() {
		      super.visitCode();
		      Label start = new Label();
		      mv.visitLabel(start);
		      mv.visitLineNumber(1, start);
		      mv.visitInsn(RETURN);
		      Label end = new Label();
		      mv.visitLabel(end);
		      mv.visitLocalVariable("this", "LTest;", null, start, end, 0);
		      mv.visitMaxs(0, 0);
		      mv.visitEnd();
		  }
		  
		  ```
		- 在上面的示例中，visitLabel用于为指令添加标签，visitLineNumber用于为标签添加行号信息。当我们反编译该代码时，会发现字节码中的每个指令都有相应的行号信息。
	- code:
		- ```java
		  line：当前正在访问的行号。
		  start：该行号对应的指令索引。
		  public void visitLineNumber(final int line, final Label start) {
		  ```
- ## 31、visitMaxs()
	- 介绍：用于通知ASM计算出方法的最大栈大小和局部变量表的最大尺寸，并设置这些信息。
	- 使用：
		- 该方法通常在MethodVisitor的visitEnd方法之前被调用，以便在方法被访问器处理完毕后为方法提供栈和局部变量表的大小信息。在方法体访问器（如MethodVisitor）中生成字节码时，需要考虑这些最大值。如果这些最大值设置得不正确，生成的字节码可能会导致VerifyError。
		- 例如，以下代码片段演示了如何在MethodVisitor的实现中调用visitMaxs方法：
			- ```java
			  public class MyMethodVisitor extends MethodVisitor {
			      
			      public MyMethodVisitor(MethodVisitor mv) {
			          super(ASM9, mv);
			      }
			      
			      public void visitCode() {
			          // 将指令添加到字节码中
			          super.visitCode();
			      }
			      
			      public void visitMaxs(int maxStack, int maxLocals) {
			          super.visitMaxs(maxStack, maxLocals);
			      }
			      
			      public void visitEnd() {
			          super.visitEnd();
			      }
			  }
			  
			  ```
	- code:
		- ```java
		  maxStack参数表示方法执行时所需的最大栈深度，即方法中使用的操作数栈的最大数量，
		  maxLocals参数表示方法的局部变量表中的变量数。
		  public void visitMaxs(final int maxStack, final int maxLocals) {
		  ```
- ## 32、visitEnd()
	- 介绍：用于通知访问结束。在此方法之后，不能再向此 MethodVisitor 实例中添加任何内容。
	- 使用：此方法没有参数，仅作为一个信号来表示该方法的访问已经完成。
	- code:
		- ```java
		  
		  
		  public void visitEnd() {
		  ```
-
-