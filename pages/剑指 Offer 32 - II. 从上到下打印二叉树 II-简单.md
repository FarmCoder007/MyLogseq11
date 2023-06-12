title:: 剑指 Offer 32 - II. 从上到下打印二叉树 II-简单

- ## 题目
	- 从上到下按层打印二叉树，同一层的节点按从左到右的顺序打印，每一层打印到一行。
	-
	- ## 例如:
	- 给定二叉树: `[3,9,20,null,null,15,7]`,
		- ```java
		     3
		     / \
		    9  20
		      /  \
		     15   7
		  ```
	- 返回其层次遍历结果：
		- ```java
		  [
		    [3],
		    [9,20],
		    [15,7]
		  ]
		  ```
- ## [思路](https://leetcode.cn/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/solutions/137255/mian-shi-ti-32-ii-cong-shang-dao-xia-da-yin-er-c-5/)
- ## 代码
	- ```java
	  class Solution {
	      public List<List<Integer>> levelOrder(TreeNode root) {
	          // 借助队列存储一层 父节点
	          Queue<TreeNode> queue = new LinkedList<>();
	          // 存储所有结果
	          List<List<Integer>> res = new ArrayList<>();
	          // 1、先将根节点 加入队列 启动循环
	          if(root != null) queue.add(root);
	          while(!queue.isEmpty()) {
	              // 存储一层的结果
	              List<Integer> tmp = new ArrayList<>();
	              for(int i = queue.size(); i > 0; i--) {
	                  TreeNode node = queue.poll();
	                  // 一层的节点取出 加入临时集合
	                  tmp.add(node.val);
	                  // 下一层的父节点加入队列 
	                  if(node.left != null) queue.add(node.left);
	                  if(node.right != null) queue.add(node.right);
	              }
	              res.add(tmp);
	          }
	          return res;
	      }
	  }
	  ```