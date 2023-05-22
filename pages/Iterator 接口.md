- # Iterator 接口是什么？
	- Iterator 接口提供了 Collection 接口的遍历。
	- ```java
	  List arrayListA = new ArrayList<>();
	  Iterator it = arrayListA.iterator();
	  while(it.hasNext()){
	    System.out.println("Value is %d", it.next());
	  }
	  
	  ```
	- **不要使用集合对象操作元素，会出现迭代器操作和list集合对象操作的并发异常**
	- **   解决办法：****list****Iterator**列表迭代器  只有list有 ，来完成在迭代中对元素进行更多的操作。   **list****Iterator**它可以完成使用迭代器对集合进行增删改查。
- # 特点
	- Iterator 只能单向遍历，但线程安全。
		- ![image-20210305104523056](https://markdown-bluestragglers.oss-cn-beijing.aliyuncs.com/img/image-20210305104523056.png)
	- Iterator 子类：ListIterator，可以双向遍历。Iterator 可以用于 Set 和 List, ListIterator 只能用于 List。