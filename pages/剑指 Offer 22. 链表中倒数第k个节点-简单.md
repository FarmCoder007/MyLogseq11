title:: 剑指 Offer 22. 链表中倒数第k个节点-简单

- # 题目
	- 输入一个链表，输出该链表中倒数第k个节点。为了符合大多数人的习惯，本题从1开始计数，即链表的尾节点是倒数第1个节点。
	- 例如，一个链表有 `6` 个节点，从头节点开始，它们的值依次是 `1、2、3、4、5、6`。这个链表的倒数第 `3` 个节点是值为 `4` 的节点。
	- # 示例
		- ```java
		  给定一个链表: 1->2->3->4->5, 和 k = 2.
		  
		  返回链表 4->5.
		  ```
- # 思路
	- ## 方法一；顺序查找
	  collapsed:: true
		- 倒数第k个，正数第 n-k+1个
		- 此时我们只需要顺序遍历完链表的 n−k 个节点即可到达倒数第 k个节点
		- - 我们首先求出链表的长度 n，然后顺序遍历到链表的第 n-k 个节点返回即可。
		- ## 代码
			- ```java
			  class Solution {
			      public ListNode getKthFromEnd(ListNode head, int k) {
			          int n = 0;
			          ListNode node = null;
			  
			          for (node = head; node != null; node = node.next) {
			              n++;
			          }
			          for (node = head; n > k; n--) {
			              node = node.next;
			          }
			  
			          return node;
			      }
			  }
			  
			  ```
		- 时间复杂度O(n) 空间O(1)
	- ## 方法二：双指针
		- 双指针：快慢指针
		- ### 思想
			- 快慢指针的思想。我们将第一个指针 fast 指向链表的第 k+1 个节点，第二个指针 slow 指向链表的第一个节点，此时指针 fast 与 slow 二者之间刚好间隔 k 个节点。此时两个指针同步向后走，当第一个指针 fast走到链表的尾部空节点时，则此时 slow 指针刚好指向链表的倒数第k个节点。
			- 方案
				- 我们首先将 fast 指向链表的头节点，然后向后走 k 步，则此时 fast 指针刚好指向链表的第 k+1 个节点。
				- 我们首先将 slow指向链表的头节点，同时 slow 与 fast 同步向后走，当 fast指针指向链表的尾部空节点时，则此时返回 slow 所指向的节点即可。
				-
		- ## 代码
			- ```java
			  class Solution {
			      public ListNode getKthFromEnd(ListNode head, int k) {
			          ListNode fast = head;
			          ListNode slow = head;
			          
			         // 将fast先走 k步，指针指向链表的k+1节点
			          while (fast != null && k > 0) {
			              fast = fast.next;
			              k--;
			          }
			          // 同时向后走，fast到达末尾，slow就是倒数第k个
			          while (fast != null) {
			              fast = fast.next;
			              slow = slow.next;
			          }
			  
			          return slow;
			      }
			  }
			  ```