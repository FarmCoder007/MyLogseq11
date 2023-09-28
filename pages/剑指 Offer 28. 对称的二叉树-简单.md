title:: 剑指 Offer 28. 对称的二叉树-简单

- ## [题目](https://leetcode.cn/problems/dui-cheng-de-er-cha-shu-lcof/)
	- 请实现一个函数，用来判断一棵二叉树是不是对称的。如果一棵二叉树和它的镜像一样，那么它是对称的。
	- 例如，二叉树 [1,2,2,3,4,4,3] 是对称的。
		- 1
		- / \
		- 2   2
		- / \ / \
		- 3  4 4  3
	- 但是下面这个 [1,2,2,null,3,null,3] 则不是镜像对称的:
		- 1
		- / \
		- 2   2
		- \   \
		- 3    3
	- ## 示例1
		- ```java
		  输入：root = [1,2,2,3,4,4,3]
		  输出：true
		  ```
	- ## 示例2
		- ```java
		  输入：root = [1,2,2,null,3,null,3]
		  输出：false
		  ```
- ## 解题思路
	- ## 对称二叉树的定义
		- 对称二叉树定义： 对于树中 任意两个对称节点 L和 R ，一定有：
			- L.val=R.val ：即此两对称节点值相等。
			- L.left.val=R.right.val：即 L的 左子节点 和 R的 右子节点 对称；
			- L.right.val=R.left.val ：即 L的 右子节点 和 R 的 左子节点 对称。
		- 根据以上规律，考虑从顶至底递归，判断每对节点是否对称，从而判断树是否为对称二叉树。
		- ![Picture1.png](https://pic.leetcode-cn.com/ebf894b723530a89cc9a1fe099f36c57c584d4987b080f625b33e228c0a02bec-Picture1.png){:height 570, :width 749}
- ## [算法流程](https://leetcode.cn/problems/dui-cheng-de-er-cha-shu-lcof/solutions/131626/mian-shi-ti-28-dui-cheng-de-er-cha-shu-di-gui-qing/)
- ## 代码
	- ```java
	  class Solution {
	      public boolean isSymmetric(TreeNode root) {
	          return root == null ? true : recur(root.left, root.right);
	      }
	      
	     // 递归判断  L.left, R.right  L.right, R.left   相等
	      boolean recur(TreeNode L, TreeNode R) {
	          // 1、左右都为null  true
	          if(L == null && R == null) return true;
	          // 有一个为null 或者 值不等 false
	          if(L == null || R == null || L.val != R.val) return false;
	          // 递归
	          return recur(L.left, R.right) && recur(L.right, R.left);
	      }
	  }
	  ```