- # 一、Object
	- Object:所有类的根类。
	- Object是不断抽取而来，具备着所有对象都具备的共性内容。
- # 二、equals
  collapsed:: true
	- ![image.png](../assets/image_1687156904453_0.png)
	- ## object默认底层调用的也是==：
		- == 比较的地址，这里 调用的==。则默认还是比较的地址
		- ```java
		      public boolean equals(Object obj) {
		          return (this == obj);
		      }
		  ```
	- ## 一般都复写
		- 一般都会覆盖此方法，根据对象的特有内容，建立判断对象是否相同的依据。
- # 三、重新equals同时也应该重新hashcode
  collapsed:: true
	- ## hashcode源码,Object的hashcode 默认调用的native方法
	  collapsed:: true
		- ```java
		      public int hashCode() {
		          return identityHashCode(this);
		      }
		  
		      // Android-changed: add a local helper for identityHashCode.
		      // Package-private to be used by j.l.System. We do the implementation here
		      // to avoid Object.hashCode doing a clinit check on j.l.System, and also
		      // to avoid leaking shadow$_monitor_ outside of this class.
		      /* package-private */ static int identityHashCode(Object obj) {
		          int lockWord = obj.shadow$_monitor_;
		          final int lockWordStateMask = 0xC0000000;  // Top 2 bits.
		          final int lockWordStateHash = 0x80000000;  // Top 2 bits are value 2 (kStateHash).
		          final int lockWordHashMask = 0x0FFFFFFF;  // Low 28 bits.
		          if ((lockWord & lockWordStateMask) == lockWordStateHash) {
		              return lockWord & lockWordHashMask;
		          }
		          return identityHashCodeNative(obj);
		      }
		  
		      @FastNative
		      private static native int identityHashCodeNative(Object obj);
		  ```
	- ![image.png](../assets/image_1687158184831_0.png)
	- hashcode协定：想等的对象必须具备想等的哈希码
	-
- # 四、getClass
  collapsed:: true
	- getClass获取当前对象，所属字节码文件对象。
	- 比如创建类，jvm先加载Class对象进堆中
	- ![image.png](../assets/image_1687160189898_0.png)
- # 五、toString
  collapsed:: true
	- 概述：默认这个
		- ![image.png](../assets/image_1687160479323_0.png)
	- ## 一般重写，输出自己想要的