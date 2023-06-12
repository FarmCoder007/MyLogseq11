title:: 剑指 Offer 55 - I. 二叉树的深度-简单

- ## 题目
	- 输入一棵二叉树的根节点，求该树的深度。从根节点到叶节点依次经过的节点（含根、叶节点）形成树的一条路径，最长路径的长度为树的深度。
	- 例如：
	- 给定二叉树 `[3,9,20,null,null,15,7]`，
		- ```java
		      3
		     / \
		    9  20
		      /  \
		     15   7
		  ```
	- 返回它的最大深度 3 。
- ## [思路](https://leetcode.cn/problems/er-cha-shu-de-shen-du-lcof/solutions/159058/mian-shi-ti-55-i-er-cha-shu-de-shen-du-xian-xu-bia/)
- ## 方法一、后续遍历
	- ```java
	  class Solution {
	      public int maxDepth(TreeNode root) {
	          if(root == null) return 0;
	          return Math.max(maxDepth(root.left), maxDepth(root.right)) + 1;
	      }
	  }
	  ```
- ## 方法二、层序遍历
	- ```java
	  class Solution {
	      public int maxDepth(TreeNode root) {
	          if(root == null) return 0;
	          List<TreeNode> queue = new LinkedList<>() {{ add(root); }}, tmp;
	          int res = 0;
	          while(!queue.isEmpty()) {
	              tmp = new LinkedList<>();
	              for(TreeNode node : queue) {
	                  if(node.left != null) tmp.add(node.left);
	                  if(node.right != null) tmp.add(node.right);
	              }
	              queue = tmp;
	              res++;
	          }
	          return res;
	      }
	  }
	  ```