- # 以 oos.writeObject(obj) 为例分析
- ## 1. ObjectOutputStream的构造函数设置enableOverride = false
	- ```java
	  public ObjectOutputStream(OutputStream out) throws IOException {
	        verifySubclass();
	        bout = new BlockDataOutputStream(out);
	        handles = new HandleTable(10, (float) 3.00);
	        subs = new ReplaceTable(10, (float) 3.00);
	        enableOverride = false;//enableOverride = false
	  		...
	  }
	  ```
- ## 2. 所以writeObject方法执行的是writeObject0(obj, false);
	- ```java
	  public final void writeObject(Object obj) throws IOException {
	      //enableOverride=false,不走这里
	      if (enableOverride) {
	          writeObjectOverride(obj);
	          return;
	      }
	      try {//一般情况都走这里
	      	writeObject0(obj, false);
	      ...
	  }
	  ```
- ## 3. 在writeObject0方法中,代码非常多，看重点
	- ```java
	  /**
	  * Underlying writeObject/writeUnshared implementation.
	  */
	  private void writeObject0(Object obj, boolean unshared) throws IOException
	      ...
	      // remaining cases
	      if (obj instanceof String) {
	      	writeString((String) obj, unshared);
	      } else if (cl.isArray()) {
	      	writeArray(obj, desc, unshared);
	      } else if (obj instanceof Enum) {
	      	writeEnum((Enum<?>) obj, desc, unshared);
	      } else if (obj instanceof Serializable) {
	          //看这里
	          writeOrdinaryObject(obj, desc, unshared);
	      } else {//如果没有实现Serializable接口，会报
	     		 NotSerializableException
	      if (extendedDebugInfo) {
	      	throw new NotSerializableException(cl.getName() + "\n" + debugInfoStack.toString());
	      } else {
	      	throw new NotSerializableException(cl.getName());
	      ...
	  }
	  ```
- ## 4. 在writeOrdinaryObject(obj, desc, unshared)方法中
	- ```java
	  private void writeOrdinaryObject(Object obj,ObjectStreamClass desc,boolean unshared)
	      ...
	      if (desc.isExternalizable() && !desc.isProxy()) {
	            //如果对象实现了Externalizable接口，那么执行
	            writeExternalData((Externalizable) obj)方法
	            writeExternalData((Externalizable) obj);
	      } else {//如果对象实现的是Serializable接口，那么执行的是
	            writeSerialData(obj, desc)
	            writeSerialData(obj, desc);
	      }
	      ...
	      }
	      //这里我们看看writeExternalData
	  ```
- ## 5. writeSerialData方法，主要执行方法：defaultWriteFields(obj, slotDesc)
	- ```java
	  /**
	  * Writes instance data for each serializable class of given object,
	  from
	  * superclass to subclass.
	  * 最终写序列化的方法
	  */
	  private void writeSerialData(Object obj, ObjectStreamClass desc)throws IOException{
	      ...
	      if (slotDesc.hasWriteObjectMethod()) {
	          //如果writeObjectMethod != null（目标类中定义了私有的writeObject方法），
	            // 那么将调用目标类中的writeObject方法
	          ...
	          slotDesc.invokeWriteObject(obj, this);
	          ...
	      } else {//如果如果writeObjectMethod == null， 那么将调用默认的defaultWriteFields方法来读取目标类中的属性
	          defaultWriteFields(obj, slotDesc);
	      }
	      }
	  }
	  ```
- ## 6. 在ObjectStreamClass中,ObjectOutputStream（ObjectInputStream）
	- 会寻找目标类中的私有的writeObject（readObject）方法，赋值给变量writeObjectMethod（readObjectMethod）
	- ```java
	  /**
	  * Creates local class descriptor representing given class.
	  */
	  private ObjectStreamClass(final Class<?> cl) {
	        ...
	        if (externalizable) {
	        		cons = getExternalizableConstructor(cl);
	        } else {//，在序列化（反序列化）的时候，
	              ObjectOutputStream（ObjectInputStream）
	              // 会寻找目标类中的私有的writeObject（readObject）方法，
	              // 赋值给变量writeObjectMethod（readObjectMethod）
	              cons = getSerializableConstructor(cl);
	              writeObjectMethod = getPrivateMethod(cl,"writeObject",
	              new Class<?>[] { ObjectOutputStream.class },Void.TYPE);
	              readObjectMethod = getPrivateMethod(cl,"readObject",
	              new Class<?>[] { ObjectInputStream.class },Void.TYPE);
	              readObjectNoDataMethod = getPrivateMethod(cl, "readObjectNoData", null, Void.TYPE);
	              hasWriteObjectData = (writeObjectMethod != null);
	        }
	        domains = getProtectionDomains(cons, cl);
	        writeReplaceMethod = getInheritableMethod( cl, "writeReplace", null, Object.class);
	        readResolveMethod = getInheritableMethod(cl, "readResolve", null, Object.class);
	        return null;
	        }
	        });
	        ...
	        }
	        //ObjectStreamClass类中的一个判断方法
	        boolean hasWriteObjectMethod() {
	        requireInitialized();
	        return (writeObjectMethod != null);
	  }
	  ```