- 方式：
	- 严格按照classFile格式添加对应数据
- 代码：toByteArray函数
  collapsed:: true
	- ```java
	  public byte[] toByteArray() { 
	  		...
	      //创建ByteVector存储对象，大小为上方计算的size
	      ByteVector result = new ByteVector(size);
	      //添加魔数，version int u4:次版本号+主版本号
	      result.putInt(0xCAFEBABE).putInt(this.version);
	      
	      /**
	      * 添加常量池数据：常量池大小u2 + 常量数组内容大小
	      * void putConstantPool(ByteVector output) {
	          output.putShort(this.constantPoolCount).putByteArray(this.constantPool.data, 0,this.constantPool.length);
	      }
	      */
	      this.symbolTable.putConstantPool(result);
	      
	      int mask = (version & 0xFFFF) < Opcodes.V1_5 ? Opcodes.ACC_SYNTHETIC : 0;
	      //添加类标识位，当前类索引，父类索引
	      result.putShort(this.accessFlags & ~mask).putShort(this.thisClass).putShort(this.superClass);
	      //添加接口长度
	      result.putShort(this.interfaceCount);
	      //添加接口数组
	      for(int i = 0; i < this.interfaceCount; ++i) {
	      result.putShort(this.interfaces[i]);
	      }
	  		
	      //添加字段长度u2
	      result.putShort(fieldsCount);
	      //循环添加字段信息
	      for(fieldWriter = this.firstField; fieldWriter != null; fieldWriter = (FieldWriter)fieldWriter.fv) {
	          fieldWriter.putFieldInfo(result);
	      }
	  		
	      //添加方法长度
	      result.putShort(methodsCount);
	      boolean hasFrames = false;
	      boolean hasAsmInstructions = false;
	  
	      for(methodWriter = this.firstMethod; methodWriter != null; methodWriter = (MethodWriter)methodWriter.mv) {
	          hasFrames |= methodWriter.hasFrames();
	          hasAsmInstructions |= methodWriter.hasAsmInstructions();
	          methodWriter.putMethodInfo(result);
	      }
	  		
	      //添加属性长度
	      result.putShort(attributesCount);
	      
	      ···
	      //添加属性表
	      if (this.firstAttribute != null) {
	          this.firstAttribute.putAttributes(this.symbolTable, result);
	      }
	  ```
- 添加数据：
  创建大小为size的字节集合对象ByteVector
  按照classFile格式由前往后依次添加元素