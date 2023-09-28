title:: 剑指 Offer 09. 用两个栈实现队列-简单

- # [题目](https://leetcode.cn/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)
	- 用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 `appendTail` 和 `deleteHead` ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，`deleteHead` 操作返回 -1 )
- # 解题思路
  collapsed:: true
	- 栈无法实现队列功能： 栈底元素（对应队首元素）无法直接删除，需要将上方所有元素出栈。
	- 双栈可实现列表倒序： 设有含三个元素的栈 A=[1,2,3] 和空栈 B=[]。若循环执行 A 元素出栈并添加入栈 B ，直到栈 A 为空，则 A=[] , B=[3,2,1] ，即 栈 B 元素实现栈 A 元素倒序 。
	- 利用栈 B 删除队首元素： 倒序后，B 执行出栈则相当于删除了 A 的栈底元素，即对应队首元素。
	- ![image.png](../assets/image_1694230455760_0.png)
	-
- # 代码
	- 实现：[[#red]]==**加入队尾appendTail() ，删除队首deleteHead()**== 两个函数
	- 因此我们可以设计栈 A 用于加入队尾操作，栈 B 用于将元素倒序，从而实现删除队首元素
	- ![image.png](../assets/image_1694230435786_0.png)
	- ```java
	  class CQueue {
	      // inStack 只存  outStack 只取 没有了 从inStack 全部取出
	      Stack<Integer> inStack;
	      Stack<Integer> outStack;
	  
	      public CQueue() {
	          inStack = new Stack<Integer>();
	          outStack = new Stack<Integer>();
	      }
	  
	      public void appendTail(int value) {
	          inStack.push(value);
	      }
	  
	      public int deleteHead() {
	          if (outStack.isEmpty()) {
	              if (inStack.isEmpty()) {
	                  return -1;
	              }
	              in2out();
	          }
	          return outStack.pop();
	      }
	  
	      // 从in 全部 取出 加入 out
	      private void in2out() {
	          while (!inStack.isEmpty()) {
	              outStack.push(inStack.pop());
	          }
	      }
	  }
	  ```