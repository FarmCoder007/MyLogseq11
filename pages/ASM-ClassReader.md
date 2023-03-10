- ## 一、作用
	- 主要负责读取.class文件里的内容，然后拆分成各个不同的部分；如何实现呢？
- ## 二、ClassReader的实现
	- 源码分析,省略了些代码
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
	- 属性分析：
		- int[] cpInfoOffsets：//数据的索引信息:标识了classFileBuffer中数据里包含的常量池位置
			- 由class文件往后读取8个字节，在classFile中可知（魔数u4，次版本号u2，主版本号u2）constant_pool_count即常量池大小为cpInfoOffsets数组大小，数组中数据为当前常量在classFile中的偏移量，用于快速获取常量；