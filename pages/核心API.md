- ## 一、组成
	- Core API包括asm.jar、asm-util.jar和asm-commons.jar
- ## 二、CoreAPI组成(在org.objectweb.asm包中)
	- ### 1、ClassReader类：
		- 字节码的读取与分析引擎。它采用类似SAX的事件读取机制，每当有事件发生时，调用注册的ClassVisitor、AnnotationVisitor、FieldVisitor、MethodVisitor做相应的处理
	- ### 2、ClassVisitor接口：
		- 定义在读取Class字节码时会触发的事件，如类头解析完成、注解解析、字段解析、方法解析等
		   解析器使ClassVisitor访问 JVMS 中定义的Class文件结构。 此类解析ClassFile内容，并为遇到的每个字段，方法和字节码指令调用给定ClassVisitor的适当访问方法。
	- ### 3、AnnotationVisitor接口：
		- 定义在解析注解时会触发的事件，如解析到一个基本值类型的注解、enum值类型的注解、Array值类型的注解、注解值类型的注解等
	- ### 4、FieldVisitor接口：
		- 定义在解析字段时触发的事件，如解析到字段上的注解、解析到字段相关的属性等。
		   MethodVisitor接口：定义在解析方法时触发的事件，如方法上的注解、属性、代码等。
	- ### 5、ClassWriter类：
		- 它实现了ClassVisitor接口，用于拼接字节码。
	- ### 6、AnnotationWriter类：
		- 它实现了AnnotationVisitor接口，用于拼接注解相关字节码
	- ### 7、FieldWriter类：
		- 它实现了FieldVisitor接口，用于拼接字段相关字节码
	- ### 8、
- ## 二、常用的asm.jar 中api(ClassReader,classVisitor,ClassWrite)
  collapsed:: true
	- 我们常用的是asm.jar中的ClassReader,classVisitor,ClassWrite这三个类，他们的关系如下：
	- ![image.png](../assets/image_1678429847944_0.png)
- ## 三、[[ASM-ClassReader]]
-