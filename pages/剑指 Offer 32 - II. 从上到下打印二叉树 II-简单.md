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
	  public List<List<Integer>> levelOrder(TreeNode root) {
	          // 1、声明存储集合
	          List<List<Integer>> ret = new ArrayList<List<Integer>>();
	          if (root == null) {
	              return ret;
	          }
	  
	          // 2、借助队列 先进先出，首先 压入root
	          Queue<TreeNode> queue = new LinkedList<TreeNode>();
	          queue.offer(root);
	  
	          // 3、循环队列 逐层压入 取出
	          while (!queue.isEmpty()) {
	              // 4、定义存储一层的集合
	              List<Integer> level = new ArrayList<Integer>();
	              // 5、本层循环个数 一层有几个节点 循环几次
	              int currentLevelSize = queue.size();
	              for (int i = 0; i < currentLevelSize; i++) {
	                  // 6、按个取出根 
	                  TreeNode node = queue.poll();
	                  level.add(node.val);
	                  // 7、加入左 右
	                  if (node.left != null) {
	                      queue.offer(node.left);
	                  }
	                  if (node.right != null) {
	                      queue.offer(node.right);
	                  }
	              }
	              // 8、遍历完 一层 添加到结果上
	              ret.add(level);
	          }
	  
	          return ret;
	      }
	  ```