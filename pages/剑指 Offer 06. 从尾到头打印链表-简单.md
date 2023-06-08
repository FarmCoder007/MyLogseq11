title:: 剑指 Offer 06. 从尾到头打印链表-简单

- # 题目
	- 输入一个链表的头节点，从尾到头反过来返回每个节点的值（[[#red]]==**用数组返回**==）。
	- ## 示例
		- ```java
		  输入：head = [1,3,2]
		  输出：[2,3,1]
		  ```
	- **限制：**
		- `0 <= 链表长度 <= 10000`
- # 思路
	- 1、创建辅助栈，将栈每个取出，再压入新栈中。即完成了反转
	- 2、再遍历新栈，取出存入数组
	-
- # 代码
	- ```java
	  /**
	   * Definition for singly-linked list.
	   * public class ListNode {
	   *     int val;
	   *     ListNode next;
	   *     ListNode(int x) { val = x; }
	   * }
	   */
	  class Solution {
	      public int[] reversePrint(ListNode head) {
	          // 新栈
	          Stack<ListNode> stack = new Stack<ListNode>();
	          ListNode temp = head;
	          // 挨个取出 加入新栈
	          while (temp != null) {
	              stack.push(temp);
	              temp = temp.next;
	          }
	          int size = stack.size();
	          // 创建数组  每个取出
	          int[] print = new int[size];
	          for (int i = 0; i < size; i++) {
	              print[i] = stack.pop().val;
	          }
	          return print;
	      }
	  }
	  ```