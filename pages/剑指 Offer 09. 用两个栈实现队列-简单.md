title:: 剑指 Offer 09. 用两个栈实现队列-简单

- # 题目
	- 用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 `appendTail` 和 `deleteHead` ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，`deleteHead` 操作返回 -1 )
- # 解题思路
  collapsed:: true
	- 栈无法实现队列功能： 栈底元素（对应队首元素）无法直接删除，需要将上方所有元素出栈。
	- 双栈可实现列表倒序： 设有含三个元素的栈 A=[1,2,3] 和空栈 B=[]。若循环执行 A 元素出栈并添加入栈 B ，直到栈 A 为空，则 A=[] , B=[3,2,1] ，即 栈 B 元素实现栈 A 元素倒序 。
	- 利用栈 B 删除队首元素： 倒序后，B 执行出栈则相当于删除了 A 的栈底元素，即对应队首元素。
	- ![Picture0.png](https://pic.leetcode-cn.com/b813bda09374058f18449b18cc6536a5b8670d5a7b65867eb65b32066c79c1ae-Picture0.png)
	-
- # 代码
	- 实现：[[#red]]==**加入队尾appendTail() ，删除队首deleteHead()**== 两个函数
	- 因此我们可以设计栈 A 用于加入队尾操作，栈 B 用于将元素倒序，从而实现删除队首元素
	- ![](https://pic.leetcode-cn.com/d7b6c80bdcbbc293f77ed3ba0faa1e58914def046f8e013c7bb24431611e3d23-Picture3.png)
	- ```java
	  class CQueue {
	      // 双栈
	      LinkedList<Integer> A, B;
	      public CQueue() {
	          A = new LinkedList<Integer>();
	          B = new LinkedList<Integer>();
	      }
	    
	      // 队尾插入: 栈底
	      public void appendTail(int value) {
	          A.addLast(value);
	      }
	      
	      // 队首删除，
	      // 当栈 B 不为空： B中仍有已完成倒序的元素，因此直接返回 B 的栈顶元素。
	      // 否则，当 A 为空： 即两个栈都为空，无元素，因此返回 −1 。
	      // 否则： 将栈 A 元素全部转移至栈 B 中，实现元素倒序，并返回栈 B 的栈顶元素。
	      public int deleteHead() {
	          if(!B.isEmpty()) return B.removeLast();
	          if(A.isEmpty()) return -1;
	          while(!A.isEmpty())
	              B.addLast(A.removeLast());
	          return B.removeLast();
	      }
	  }
	  ```