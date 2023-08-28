## 1. getByteCount()
	- getByteCount()方法是在API12加入的，代表存储Bitmap的色素需要的最少内存。API19开始
	  getAllocationByteCount()方法代替了getByteCount()。
- ## 2. getAllocationByteCount()
	- API19之后，Bitmap加了一个Api：getAllocationByteCount()；代表在内存中为Bitmap分配的内存大
	  小。
	- ```java
	      public final int getAllocationByteCount() {
	          if (mBuffer == null) {
	  //mBuffer代表存储Bitmap像素数据的字节数组。
	              return getByteCount();
	          }
	          return mBuffer.length;
	      }
	  ```
- ## 3. getByteCount()与getAllocationByteCount()的区别
	- 一般情况下两者是相等的
	- 通过复用Bitmap来解码图片，如果被复用的Bitmap的内存比待分配内存的Bitmap大,那么
	- getByteCount()表示[[#red]]==**新解码图片占用内存的大小**==（并非实际内存大小,实际大小是复用的那个
	  Bitmap的大小），
	- getAllocationByteCount()表示[[#red]]==**被复用Bitmap真实占用的内存大小**==（即mBuffer
	  的长度）