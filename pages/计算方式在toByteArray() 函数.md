- 计算方式：
  collapsed:: true
	- 1、必要位： 由classFile格式可知总计有24个字节，接口数据有 2*interfaceCount
	- 2、其他位： 依次计算剩下的常量池，字段，方法，属性大小
	- 3、汇总以上数据获取.class文件大小
- 代码片段classWriter中的toByteArray()
  collapsed:: true
	- ```java
	  public byte[] toByteArray() {
	  
	     /**
	     * 24: u4 magic , 10个必须字段u2（minor_version, major_version, constant_pool_count, access_flags,
	     *    this_class , super_class, interfaces_count, fields_count, methods_count, attributes_count）
	     * 接口字段集合为 u2 * interfaceCount
	     * 剩余未计算： cp_info , field_info , method_info , attribute_info
	     */
	      int size = 24 + 2 * this.interfaceCount;
	      int fieldsCount = 0;
	  		
	     /**
	     * 链表计算字段占用大小 field_info
	     */
	      FieldWriter fieldWriter;
	      for(fieldWriter = this.firstField; fieldWriter != null; fieldWriter = (FieldWriter)fieldWriter.fv) {
	          ++fieldsCount;
	          size += fieldWriter.computeFieldInfoSize();
	      }
	  
	     /**
	     * 链表计算方法占用大小 method_info
	     */
	      int methodsCount = 0;
	      MethodWriter methodWriter;
	      for(methodWriter = this.firstMethod; methodWriter != null; methodWriter = (MethodWriter)methodWriter.mv)      {
	          ++methodsCount;
	          size += methodWriter.computeMethodInfoSize();
	      }
	      
	     /**
	     * 计算属性表占用大小 attribute_info
	     */
	      int attributesCount = 0;
	      ......
	  
	      if (firstAttribute != null) {
	          attributesCount += firstAttribute.getAttributeCount();
	          size += firstAttribute.computeAttributesSize(symbolTable);
	      }
	     /**
	     * 计算常量池占用大小 cp_info
	     */
	      size += this.symbolTable.getConstantPoolLength();
	  ```
	-