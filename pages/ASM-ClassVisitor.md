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
	- ### 4、visitNestHost()：
		- ```java
		  用于访问当前类的嵌套类。该方法的作用是通知 ClassVisitor 此类是一个嵌套类，
		  并提供嵌套类的宿主类信息。
		  //nestHost:
		  public void visitNestHost(final String nestHost) {
		  ```