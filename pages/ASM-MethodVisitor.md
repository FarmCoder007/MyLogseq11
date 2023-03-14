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
	- 介绍：用于访问方法上的注解信息。
	- 使用：
		- 在 Java 中，注解可以用于方法上，用于给方法添加一些元数据信息。在 ASM 中，可以通过 visitAnnotation 方法来访问方法上的注解信息。
		- visitAnnotation 方法会在访问方法时被调用，用于访问方法上的注解信息。其中，descriptor 参数表示注解的类型描述符，visible 参数表示注解是否可见。在访问注解信息时，需要返回一个 AnnotationVisitor 对象，用于访问注解的元素信息。
		- 需要注意的是，如果注解不可见，则在使用反射 API 获取注解信息时可能无法获取到该注解。如果需要在运行时获取注解信息，建议将注解设置为可见。
		- 例如，如果方法上有 @MyAnnotation("value") 注解，则可以通过以下代码来访问该注解的信息：
		- ```java
		  ```
	- code:
		- ```java
		  descriptor：表示注解的类型描述符，以字符串形式给出。
		  例如，对于 @MyAnnotation 注解，其类型描述符为 "Lcom/example/MyAnnotation;"。
		  visible：表示注解是否可见，以布尔值形式给出。
		  AnnotationVisitor visitAnnotation(String descriptor, boolean visible);
		  ```
- ## 4、visitTypeAnnotation()
- ## 5、visitAnnotableParameterCount()
- ## 6、visitParameterAnnotation()
- ## 7、visitAttribute()
- ## 8、visitCode()
- ## 9、visitFrame()
- ## 10、visitInsn()
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