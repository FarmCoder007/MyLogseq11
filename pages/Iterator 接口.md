- # Iterator 接口是什么？
	- Iterator 接口提供了 Collection 接口的遍历。
	- ```java
	  List arrayListA = new ArrayList<>();
	  Iterator it = arrayListA.iterator();
	  while(it.hasNext()){
	    System.out.println("Value is %d", it.next());
	  }
	  
	  ```
- # 特点
	- Iterator 只能单向遍历，但线程安全。
		- ![image-20210305104523056](https://markdown-bluestragglers.oss-cn-beijing.aliyuncs.com/img/image-20210305104523056.png)
	- Iterator 子类：ListIterator，可以双向遍历。Iterator 可以用于 Set 和 List, ListIterator 只能用于 List。