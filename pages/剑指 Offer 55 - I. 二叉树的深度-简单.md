title:: 剑指 Offer 55 - I. 二叉树的深度-简单

- ## [题目](https://leetcode.cn/problems/er-cha-shu-de-shen-du-lcof/)
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
- ###### 方法一、后续遍历
  collapsed:: true
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
	  public int maxDepth(TreeNode root) {
	          // 1、容错
	          if(root == null){
	              return 0;
	          }
	          // 2、记录个数
	          int result = 0;
	          // 3、声明队列加入root
	          Queue<TreeNode> queue = new LinkedList<>();
	          queue.offer(root);
	  
	          // 4、遍历队列
	          while (!queue.isEmpty()){
	              // 声明层Count
	              int levelCount = queue.size();
	              // 该层有多少个节点，遍历多少次
	              while (levelCount > 0){
	                  TreeNode levelRoot = queue.poll();
	                  if(levelRoot.left != null){
	                      queue.offer(levelRoot.left);
	                  }
	                  if(levelRoot.right != null){
	                      queue.offer(levelRoot.right);
	                  }
	                  levelCount -- ;
	              }
	              // 每遍历一层 深度数++
	              result ++;
	          }
	          return result;
	      }
	  ```