title:: 剑指 Offer 25. 合并两个排序的链表-简单

- # [题目](https://leetcode.cn/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)
	- 输入两个递增排序的链表，合并这两个链表并使新链表中的节点仍然是递增排序的。
	- ## 示例
		- ```java
		  输入：1->2->4, 1->3->4
		  输出：1->1->2->3->4->4
		  ```
	- **限制：**
		- `0 <= 链表长度 <= 1000`
- # 思路
	- ## 方法一：迭代
		- 我们可以用迭代的方法来实现上述算法。当 l1 和 l2 都不是空链表时，判断 l1 和 l2 哪一个链表的头节点的值更小，将较小值的节点添加到结果里，当一个节点被添加到结果里之后，将对应链表中的节点向后移一位。
		- ## 算法
			- 1、首先，我们设定一个哨兵节点 `prehead` ，这可以在最后让我们比较容易地返回合并后的链表。
			- 2、我们维护一个 prev 指针，我们需要做的是调整它的 next 指针。
			- 3、然后，我们重复以下过程，直到 l1 或者 l2 指向了 null ：
				- 如果 l1 当前节点的值小于等于 l2 ，我们就把 l1 当前的节点接在 prev 节点的后面同时将 l1 指针往后移一位。
				- 否则，我们对 l2 做同样的操作。不管我们将哪一个元素接在了后面，我们都需要把 prev 向后移一位
			- 4、在循环终止的时候， l1 和 l2 至多有一个是非空的。由于输入的两个链表都是有序的，所以不管哪个链表是非空的，它包含的所有元素都比前面已经合并链表中的所有元素都要大。这意味着我们只需要简单地将非空链表接在合并链表的后面，并返回合并链表即可。
			-
		- ## 代码
			- ```java
			  class Solution {
			      public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
			          // 建立哨兵节点，作为返回链表头
			          ListNode prehead = new ListNode(-1);
			          // 维护一个前指针
			          ListNode prev = prehead;
			          // 
			          while (l1 != null && l2 != null) {
			              // 取l1最小值  赋值
			              if (l1.val <= l2.val) {
			                  prev.next = l1;
			                  l1 = l1.next;
			              } else {
			                  prev.next = l2;
			                  l2 = l2.next;
			              }
			              // 指针后移
			              prev = prev.next;
			          }
			  
			          // 合并后 l1 和 l2 最多只有一个还未被合并完，我们直接将链表末尾指向未合并完的链表即可
			          prev.next = l1 == null ? l2 : l1;
			  
			          return prehead.next;
			      }
			  }
			  ```
		- ## 复杂度
			- 时间：O(M+N)
			- 空间：O(1)