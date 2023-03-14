- ## 一、介绍
	- ASM库中的Opcodes类提供了Java字节码指令的常量表示，可用于在访问方法和访问器中生成指令。下面是一些常用的指令：
- ## 二、常用指令
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