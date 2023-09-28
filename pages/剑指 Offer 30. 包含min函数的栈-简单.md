title:: 剑指 Offer 30. 包含min函数的栈-简单

- ## [题目](https://leetcode.cn/problems/bao-han-minhan-shu-de-zhan-lcof/)
	- 定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的 min 函数在该栈中，调用 min、push 及 pop 的时间复杂度都是 O(1)。
	- ## 示例
		- ```java
		  MinStack minStack = new MinStack();
		  minStack.push(-2);
		  minStack.push(0);
		  minStack.push(-3);
		  minStack.min();   --> 返回 -3.
		  minStack.pop();
		  minStack.top();      --> 返回 0.
		  minStack.min();   --> 返回 -2.
		  ```
	- - 各函数的调用总次数不超过 20000 次
- # 思路：
  collapsed:: true
	- >普通栈的 push() 和 pop() 函数的复杂度为 O(1) ；而获取栈最小值 min() 函数需要遍历整个栈，复杂度为 O(N) 。
	- 1、所以需要借助辅助栈（每个元素存储当前节点下最小值）
	- 2、普通栈压入一个元素，辅助栈，同时压入此时与新元素对比下的最小值。（保证2个栈元素数量相同，只不过辅助栈存的都是对应压入那个元素时刻的最小值）
	- 3、出栈时，同时出栈
	- ![辅助栈.gif](../assets/辅助栈_1686144391149_0.gif)
	-
- # 代码
	- ```java
	  class MinStack {
	      // 普通栈
	      LinkedList<Integer> xStack;
	      // 辅助栈 也是最小栈
	      LinkedList<Integer> minStack;
	  
	      public MinStack() {
	          xStack = new LinkedList<Integer>();
	          minStack = new LinkedList<Integer>();
	          minStack.push(Integer.MAX_VALUE);
	      }
	      
	      public void push(int x) {
	          // 压入普通元素
	          xStack.push(x);
	          // 压入最小元素
	          minStack.push(Math.min(minStack.peek(), x));
	      }
	      
	     // 出栈一起出栈
	      public void pop() {
	          xStack.pop();
	          minStack.pop();
	      }
	      
	      public int top() {
	          return xStack.peek();
	      }
	      
	      public int min() {
	          return minStack.peek();
	      }
	  }
	  ```