title:: 剑指 Offer 54. 二叉搜索树的第k大节点-简单

- ## [题目](https://leetcode.cn/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/)
	- 给定一棵二叉搜索树，请找出其中第 `k` 大的节点的值。
	- ## 示例
	  collapsed:: true
		- ```java
		  输入: root = [3,1,4,null,2], k = 1
		     3
		    / \
		   1   4
		    \
		     2
		  输出: 4
		  ```
	- ## 示例
	  collapsed:: true
		- ```java
		  输入: root = [5,3,6,2,4,null,null,1], k = 3
		         5
		        / \
		       3   6
		      / \
		     2   4
		    /
		   1
		  输出: 4
		  ```
- ## [思路](https://leetcode.cn/problems/er-cha-sou-suo-shu-de-di-kda-jie-dian-lcof/solutions/184216/mian-shi-ti-54-er-cha-sou-suo-shu-de-di-k-da-jie-d/)
	- 根据以上性质，易得二叉搜索树的 中序遍历倒序 为 递减序列 。
	- 因此，求 “二叉搜索树第 k 大的节点” 可转化为求 “此树的中序遍历倒序的第 k 个节点
	- ![image.png](../assets/image_1694868453037_0.png)
- ## 代码
	- ```java
	  1、中序遍历的 倒序遍历 右根左  --k = 0 赋值result
	  class Solution {
	      // 1、定义返回结果 和 循环k
	      int result, k;
	      public int kthLargest(TreeNode root, int k) {
	          this.k = k;
	          dfs(root);
	          return result;
	      }
	      void dfs(TreeNode root) {
	          if(root == null) return;
	          if(k == 0) return;
	          // 右遍历
	          dfs(root.right);
	          // 根
	          if(--k == 0) result = root.val;
	          // 左遍历
	          dfs(root.left);
	      }
	  }
	  
	  k=3时
	  1、dfs传入 根节点5 
	  	  右遍历 dfs传入6	走一次--k   k = 2
	  	  走到5根   	      走一次--k   k = 1
	    	  左遍历 dfs传入3
	        		 右遍历 dfs传入4 走一次--k   k = 0   所以输出4
	  
	  ```