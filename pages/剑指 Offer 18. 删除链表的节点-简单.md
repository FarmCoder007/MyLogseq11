title:: 剑指 Offer 18. 删除链表的节点-简单

- # 题目
	- 给定单向链表的头指针和一个要删除的节点的值，定义一个函数删除该节点。
	- 返回删除后的链表的头节点。
- ## 示例 1:
	- ```
	  **输入:** head = [4,5,1,9], val = 5
	  **输出:** [4,1,9]
	  **解释: **给定你链表中值为 5 的第二个节点，那么在调用了你的函数之后，该链表应变为 4 -> 1 -> 9.
	  ```
- ## 示例2
	- ```java
	  输入: head = [4,5,1,9], val = 1
	  输出: [4,5,9]
	  解释: 给定你链表中值为 1 的第三个节点，那么在调用了你的函数之后，该链表应变为 4 -> 5 -> 9.
	  ```
- ## 说明
	- 题目保证链表中节点的值互不相同
- # 思路[地址](https://leetcode.cn/problems/shan-chu-lian-biao-de-jie-dian-lcof/solutions/167212/mian-shi-ti-18-shan-chu-lian-biao-de-jie-dian-sh-2/)
  collapsed:: true
	- 1、定位节点： 遍历链表，直到 head.val == val 时跳出，即可定位目标节点。
	- 2、修改引用： 设节点 cur 的前驱节点为 pre ，后继节点为 cur.next ；则执行 pre.next = cur.next ，即可实现删除 cur 节点。
	- ![Picture1.png](https://pic.leetcode-cn.com/1613757478-NBOvjn-Picture1.png)
- # 代码
	- ```java
	  class Solution {
	      public ListNode deleteNode(ListNode head, int val) {
	          // 首个就找到直接返回下一个节点
	          if(head.val == val) return head.next;
	          
	          // 定义双指针
	          ListNode pre = head, cur = head.next;
	          // 终止条件 链表遍历完 || 找到值
	          while(cur != null && cur.val != val) {
	              pre = cur;
	              cur = cur.next;
	          }
	          // 删除找到的curr节点
	          if(cur != null) pre.next = cur.next;
	          return head;
	      }
	  }
	  
	  ```
- # 复杂度
	- 时间O(n)  空间O(1)