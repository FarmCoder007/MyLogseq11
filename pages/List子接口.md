- [list对象复制](https://blog.csdn.net/qq_40542534/article/details/112566277)
  title:: List子接口
- ## list的常用方法
	- 1，添加
	  	void add(index,element);
	  	void add(index,collection);
	- 2，删除；
	  	Object remove(index):
	- 3，修改：
	  	Object set(index,element);
	- 4，获取：
	  	Object get(index);
	  	int indexOf(object);
	  	int lastIndexOf(object);
	  	List subList(from,to);
	  list集合是可以完成对元素的增删改查。
- LinkList [实现方式可视为队列]
  collapsed:: true
	- Queue中 add/offer[添加]，element/peek[取元素]，remove/poll[删除]区别
		- add/offer[添加]
			- add       增加一个元索     如果队列已满，则抛出一个IIIegaISlabEepeplian异常
			- offer     添加一个元素并返回true        如果队列已满，则返回false [容错]
		- element/peek[取元素]
			- element  返回队列头部的元素   如果队列为空，则抛出一个NoSuchElementException异常
			- peek       返回队列头部的元素              如果队列为空，则返回null
		- remove/poll[删除]
			- remove   移除并返回队列头部的元素     如果队列为空，则抛出一个NoSuchElementException异常
			- poll         移除并返问队列头部的元素     如果队列为空，则返回null
		-
	-
	-
- [list for 循环报错 ConcurrentModificationException](https://blog.csdn.net/J_bean/article/details/114899014)
	- 解决：
		- [ConcurrentLinkedQueue](https://blog.csdn.net/z69183787/article/details/81064982)
-