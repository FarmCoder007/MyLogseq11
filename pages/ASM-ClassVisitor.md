- 一、
- ## 二、相关api
- ## 三、api调用顺序，从上到下
	- ### 1、visit( ) ：访问类的开始的部分。
	  collapsed:: true
		- ```java
		  // version:类版本。次要版本存储在16个最高有效位中，主要版本存储在十六个最低有效位中
		  // access: 类的访问标志（参见Opcodes）。此参数还指示该类是否已弃用。
		  // name:表示类的全限定名
		  // signature:这个类的签名。如果类不是泛型类，并且不扩展或实现泛型类或接口，则可以为null。
		  // superName:表示父类的全限定名,对于接口，超级类是Object。可以为空，但仅适用于Object类。
		  // interfaces表示实现的接口的全限定名数组。
		  public void visit(
		        final int version,
		        final int access,
		        final String name,
		        final String signature,
		        final String superName,
		        final String[] interfaces)
		  ```
	- ### 2、 visitSource()：访问源文件名和源文件的调试信息
	  collapsed:: true
		- ```java
		  // source:从中编译类的源文件的名称。可以为空
		  // debug:用于计算类的源元素和编译元素之间的对应关系的附加调试信息。可以为空。
		  public void visitSource(final String source, final String debug)
		  ```
	- ### 3、visitModule()：访问与该类对应的模块。
	  collapsed:: true
		- ```java
		  // name:模块的完全限定名称（使用圆点）
		  // access:ACC_OPEN、ACC_SYNTHETIC和ACC_MANDATED中的模块访问标志。
		  // version:模块版本或空。
		  public ModuleVisitor visitModule(final String name, final int access, final String version) 
		  ```
	- ### 4、visitNestHost()：用于访问当前类的嵌套类
	  collapsed:: true
		- 嵌套类是从 Java 11 开始引入的一种新的类类型，它允许在一个类中定义另一个类。嵌套类可以是静态的或非静态的，它们可以访问宿主类的私有成员
		- ```java
		  用于访问当前类的嵌套类。该方法的作用是通知 ClassVisitor 此类是一个嵌套类，
		  并提供嵌套类的宿主类信息。
		  //nestHost: 表示当前类的宿主类的名称。如果当前类不是嵌套类，则该参数应为 null
		  public void visitNestHost(final String nestHost) {
		  ```
	- ### 5、visitOuterClass()：访问外部类的信息，即包含该类的最外层类的信息。
	  collapsed:: true
		- 作用：
			- 访问一个类的外部类信息。该方法的作用是通知 ClassVisitor 该类的外部类信息，即包含该类的最外层类的信息。
		- ```java
		  // owner:表示包含该类的最外层类的全限定名。如果该类不是内部类，则该参数应为 null。
		  // name:表示包含该类的最外层类的简单名称。如果该类不是内部类，则该参数应为 null。
		  // descriptor:表示包含该类的最外层类的类型描述符。如果该类不是内部类，则该参数应为 null。
		  public void visitOuterClass(final String owner, final String name, final String descriptor) 
		  ```
	- ### 6、visitAnnotation()：访问类的注解
	  collapsed:: true
		- ```java
		  // descriptor:表示注解的类型描述符，以字符串形式给出。
		  // visible:表示该注解是否在运行时可见。如果该注解在运行时不可见，则应将其设置为 false。
		  public AnnotationVisitor visitAnnotation(final String descriptor, final boolean visible) {
		  ```
	- ### 7、visitTypeAnnotation()：它用于访问一个类的类型注解信息
	  collapsed:: true
		- ```java
		  
		  typeRef：表示类型注解的目标类型，以一个整数形式给出。具体来说，它表示一个指向字节码中某个类型的引用。在 Java 字节码中，类型引用是一个复杂的概念，它可以表示类的继承关系、方法的参数和返回值类型、字段的类型等。
		  typePath：表示类型注解的路径，以 TypePath 对象形式给出。类型路径是一个用于描述类型注解的位置信息的对象，它可以表示类型引用的具体位置，如类的继承链中的哪个位置、方法参数的哪个位置等。
		  descriptor：表示注解的类型描述符，以字符串形式给出。
		  visible：表示该注解是否在运行时可见。如果该注解在运行时不可见，则应将其设置为 false。
		  public AnnotationVisitor visitTypeAnnotation(
		        final int typeRef, final TypePath typePath, final String descriptor, final boolean visible)
		  ```
	- ### 8、visitAttribute():它用于访问一个类的属性信息。
	  collapsed:: true
		- 在 ASM 中，当访问一个类时，除了类的注解和类型注解外，还可能有一些其他的信息，如常量池、源文件、行号表等。这些信息都被视为类的属性信息，并使用 Attribute 对象来表示。当 ASM 访问一个类的属性信息时，会调用 visitAttribute 方法通知 ClassVisitor 属性信息，并将 Attribute 对象作为参数传入。然后，ClassVisitor 可以根据不同的属性类型进行处理，如解析常量池中的字符串、获取源文件名等。
		- 需要注意的是，如果一个类有多个属性信息，则 visitAttribute 方法会被多次调用，每次调用时传入不同的 Attribute 对象。
		- ```java
		  attribute：表示类的属性信息，以 Attribute 对象形式给出。
		  Attribute 是一个抽象类，它的子类分别表示不同类型的属性，如常量池、源文件、行号表等。
		  public void visitAttribute(final Attribute attribute)
		  ```
	- ### 9、visitNestMember()：它用于访问一个类的嵌套成员信息
	  collapsed:: true
		- 介绍：
			- 在 Java 11 中，引入了一种新的类文件结构，称为嵌套类属性（Nest Attributes）。该属性可以用来表示一个类是另一个类的嵌套类，或者一个类是一个嵌套类中的成员。当一个类被声明为嵌套类或成员时，它会被添加一个 NestHost 或 NestMembers 属性，以标识其所属的嵌套类或嵌套成员。当访问一个类时，如果该类被声明为嵌套类或嵌套成员，则 ASM 会调用 visitNestMember 方法通知 ClassVisitor 该类所属的外部类的类名，并进行处理。
			- 需要注意的是，如果一个类既是嵌套类又是嵌套成员，则可能同时具有 NestHost 和 NestMembers 属性。在这种情况下，ASM 会先调用 visitNestHost 方法处理 NestHost 属性，然后再调用 visitNestMember 方法处理 NestMembers 属性。
		- ```java
		  nestMember：表示该类所属的外部类的类名，以字符串形式给出。
		  public void visitNestMember(final String nestMember) {
		  ```
	- ### 10、visitInnerClass()：它用于访问一个类的内部类信息
	  collapsed:: true
		- 在 Java 中，内部类是指被定义在另一个类内部的类。与常规的类不同，内部类可以访问其外部类的私有成员，可以用来实现一些特殊的功能。当使用 ASM 访问一个类时，可能需要访问该类的内部类信息，例如在生成字节码时需要访问其内部类的访问修饰符。
		- visitInnerClass 方法会在访问一个类时被调用，用于通知 ClassVisitor 该类的内部类信息。其中，name 参数表示内部类的名称，outerName 参数表示外部类的名称，innerName 参数表示内部类的简单名称，access 参数表示内部类的访问修饰符。ClassVisitor 可以根据这些信息进行处理，例如生成内部类的字节码。需要注意的是，如果一个类有多个内部类，则 visitInnerClass 方法会被多次调用，每次调用时传入不同的内部类信息。
		- ```java
		  name：表示内部类的名称，以字符串形式给出。对于非内部类，该参数值为类名。
		  outerName：表示外部类的名称，以字符串形式给出。如果该类不是内部类，则该参数值为 null。
		  innerName：表示内部类的简单名称，以字符串形式给出。如果该类不是内部类，则该参数值为 null。
		  access：表示内部类的访问修饰符，以整数形式给出。
		  public void visitInnerClass(
		        final String name, final String outerName, final String innerName, final int access) 
		  ```
	- ### 11、visitField():
		- ```java
		  ```
-