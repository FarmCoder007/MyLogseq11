title:: 剑指 Offer 27. 二叉树的镜像-简单

- ## 题目
	- 请完成一个函数，输入一个二叉树，该函数输出它的镜像。
	- 例如输入：
		- ```java
		       4
		  
		     /   \
		  
		    2     7
		  
		   / \   / \
		  
		  1   3 6   9
		  ```
	- ```java
	       4
	  
	     /   \
	  
	    7     2
	  
	   / \   / \
	  
	  9   6 3   1
	  ```
	- **示例 1：**
	- ```
	  **输入：**root = [4,2,7,1,3,6,9]
	  **输出：**[4,7,2,9,6,3,1]
	  ```
- ## [思路](https://leetcode.cn/problems/er-cha-shu-de-jing-xiang-lcof/solutions/)
- ## 方法一：递归法
	- ```java
	  class Solution {
	      public TreeNode mirrorTree(TreeNode root) {
	          if(root == null) return null;
	          // 1、定义辅助  递归翻转左右
	          TreeNode tmp = root.left;
	          root.left = mirrorTree(root.right);
	          root.right = mirrorTree(tmp);
	          return root;
	      }
	  }
	  ```
- ## 方法二：辅助栈（或队列）
	- 利用栈（或队列）遍历树的所有节点 node ，并交换每个 node的左 / 右子节点。
	- ```java
	  class Solution {
	      public TreeNode mirrorTree(TreeNode root) {
	          if(root == null) return null;
	          Stack<TreeNode> stack = new Stack<>() {{ add(root); }};
	          while(!stack.isEmpty()) {
	              TreeNode node = stack.pop();
	              // 存入每层左右节点 下次交换
	              if(node.left != null) stack.add(node.left);
	              if(node.right != null) stack.add(node.right);
	              // 交互左右节点
	              TreeNode tmp = node.left;
	              node.left = node.right;
	              node.right = tmp;
	          }
	          return root;
	      }
	  }
	  ```