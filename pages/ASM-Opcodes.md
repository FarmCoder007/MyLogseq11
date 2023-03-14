- ## 一、介绍
	- Opcodes 是 ASM 框架中的一个常量接口，包含了 JVM 中的指令、操作码和一些常量等信息。以下是 Opcodes 接口中的常量列表及其作用：
- ## 1、指令码常量
  collapsed:: true
	- ACC_PUBLIC：public 属性
	  ACC_PRIVATE：private 属性
	  ACC_PROTECTED：protected 属性
	  ACC_FINAL：final 属性
	  ACC_STATIC：static 属性
	  ACC_SYNCHRONIZED：synchronized 属性
	  ACC_VOLATILE：volatile 属性
	  ACC_BRIDGE：bridge 属性
	  ACC_VARARGS：varargs 属性
	  ACC_NATIVE：native 属性
	  ACC_ABSTRACT：abstract 属性
	  ACC_STRICT：strict 属性
	  ACC_SYNTHETIC：synthetic 属性
	  ACC_ENUM：enum 属性
	  ACC_ANNOTATION：annotation 属性
	  ACC_SUPER：super 属性
	  ACC_INTERFACE：interface 属性
	  ACC_MANDATED：mandated 属性
- ## 2、JVM 操作码常量
  collapsed:: true
	- AALOAD：将 int 数组指定索引位置的值加载到栈顶
	  AASTORE：将栈顶引用类型数值存入指定数组的指定索引位置
	  ACONST_NULL：将 null 压入栈顶
	  ALOAD：将指定局部变量的引用类型值推送到栈顶
	  ANEWARRAY：创建一个引用类型的数组，并将其引用值压入栈顶
	  ARETURN：从当前方法返回引用类型数据
	  ARRAYLENGTH：获取数组长度
	  ASTORE：将栈顶引用类型数值存入指定局部变量
	  ATHROW：抛出栈顶异常对象
	  BALOAD：将 boolean 或 byte 数组指定索引位置的值加载到栈顶
	  BASTORE：将栈顶int数值存入指定byte或boolean类型数组的指定索引位置
	  BIPUSH：将单字节的常量值(-128~127)推送至栈顶
	  CALOAD：将 char 数组指定索引位置的值加载到栈顶
	  CASTORE：将栈顶int数值存入指定char类型数组的指定索引位置
	  CHECKCAST：检查类型转换，栈顶值必须是引用类型，否则抛出 ClassCastException
	  D2F：将栈顶 double 类型数值转换成 float 类型数值
	  D2I：将栈顶 double 类型数值转换成 int 类型数值
	  D2L：将栈顶 double 类型数值转换成 long 类型数值
	  DADD：将栈顶两个 double 类型数值相加并将结果压入栈顶
	  DALOAD：将 double 数组指定索引位置的值加载到栈顶
	  DASTORE：将栈顶 double 类型数值存入指定数组的指定索引位置
	  DCMPG：比较栈顶两个 double 类型数值大小，并将比较结果压入栈顶
	  DCMPL：比较栈顶
- ## 3、常用指令
  collapsed:: true
	- 加载和存储指令
	- ILOAD, ISTORE: 将int类型从本地变量表中加载到操作数栈中或从操作数栈中存储到本地变量表中
	  ALOAD, ASTORE: 将引用类型从本地变量表中加载到操作数栈中或从操作数栈中存储到本地变量表中
	  LLOAD, LSTORE: 将long类型从本地变量表中加载到操作数栈中或从操作数栈中存储到本地变量表中
	  FLOAD, FSTORE: 将float类型从本地变量表中加载到操作数栈中或从操作数栈中存储到本地变量表中
	  DLOAD, DSTORE: 将double类型从本地变量表中加载到操作数栈中或从操作数栈中存储到本地变量表中
	  算术指令
	- IADD, ISUB, IMUL, IDIV, IREM: 对int类型进行加、减、乘、除、求余运算
	  LADD, LSUB, LMUL, LDIV, LREM: 对long类型进行加、减、乘、除、求余运算
	  FADD, FSUB, FMUL, FDIV, FREM: 对float类型进行加、减、乘、除、求余运算
	  DADD, DSUB, DMUL, DDIV, DREM: 对double类型进行加、减、乘、除、求余运算
	  类型转换指令
	- I2L, L2I: 将int类型转换为long类型，或将long类型转换为int类型
	  I2F, F2I: 将int类型转换为float类型，或将float类型转换为int类型
	  I2D, D2I: 将int类型转换为double类型，或将double类型转换为int类型
	  L2F, F2L: 将long类型转换为float类型，或将float类型转换为long类型
	  L2D, D2L: 将long类型转换为double类型，或将double类型转换为long类型
	  F2D, D2F: 将float类型转换为double类型，或将double类型转换为float类型
	  控制流程指令
	- IFEQ, IFNE, IFGT, IFGE, IFLT, IFLE: 对int类型进行比较，如果满足条件，则跳转到指定的代码位置
	  GOTO: 无条件跳转到指定的代码位置
	  RETURN: 从当前方法返回
- ## ASM 指令示例：
	- 1、Opcodes.GETSTATIC：
	  collapsed:: true
		- 介绍：是 ASM 中的一个指令，用于访问类的静态变量。
		- 使用：
			- 其操作数栈不会有变化，它会将指定类的指定静态变量的值压入栈中。其参数如下：
			- owner: 静态变量所在类的内部名称（字符串类型）。
			  name: 静态变量的名称（字符串类型）。
			  descriptor: 静态变量的类型（字符串类型）。
			  使用场景：GETSTATIC 指令通常在访问类的静态变量时使用。例如，在以下 Java 代码中：
			- ```java
			  public class Example {
			      public static final int EXAMPLE_VALUE = 42;
			  }
			  
			  ```
			- 要在字节码中访问 EXAMPLE_VALUE 静态变量，可以使用以下 ASM 代码：
			- ```java
			  methodVisitor.visitFieldInsn(Opcodes.GETSTATIC, "Example", "EXAMPLE_VALUE", "I");
			  
			  ```
			- 此代码使用 Opcodes.GETSTATIC 指令，访问 Example 类中的 EXAMPLE_VALUE 静态变量。在这个例子中，owner 参数的值为 "Example"，name 参数的值为 "EXAMPLE_VALUE"，descriptor 参数的值为 "I"（整数类型）。
	- 2、Opcodes.PUTSTATIC：
		- 介绍：
			- 将一个静态字段（类字段）的值从操作数栈顶弹出并存储到指定的类中。其语法为：
		- 使用场景：
			- 1、用于为一个静态字段赋值。
				- ```java
				  PUTSTATIC owner name desc
				  
				  ```
				- ```java
				    public static int count = 0;
				  
				  public static void main(String[] args) {
				      count = 1;
				  }
				  上述示例代码中，使用了 PUTSTATIC 指令为 count 静态字段赋值。
				  ```
			- 2、用于加载一个类的静态字段。
				- GETSTATIC owner name desc
				- ```java
				  public static int count = 0;
				  
				  public static void main(String[] args) {
				      int n = count;
				  }
				  
				  ```