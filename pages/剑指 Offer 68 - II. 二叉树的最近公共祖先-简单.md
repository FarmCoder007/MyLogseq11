title:: 剑指 Offer 68 - II. 二叉树的最近公共祖先-简单

- ## [题目](https://leetcode.cn/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/description/)
	- 给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。
	- [百度百科](https://baike.baidu.com/item/%E6%9C%80%E8%BF%91%E5%85%AC%E5%85%B1%E7%A5%96%E5%85%88/8918834?fr=aladdin)中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。”
	- 例如，给定如下二叉树:  root = [3,5,1,6,2,0,8,null,null,7,4]
		- ![](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/15/binarytree.png)
- ## **示例 1:**
	- ```java
	  输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
	  输出: 3
	  解释: 节点 5 和节点 1 的最近公共祖先是节点 3。
	  ```
- ## **示例 2:**
	- ```java
	  输入: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 4
	  输出: 5
	  解释: 节点 5 和节点 4 的最近公共祖先是节点 5。因为根据定义最近公共祖先节点可以为节点本身。
	  ```
- ## [思路](https://leetcode.cn/problems/er-cha-shu-de-zui-jin-gong-gong-zu-xian-lcof/solutions/217281/mian-shi-ti-68-ii-er-cha-shu-de-zui-jin-gong-gon-7/)
- ## 代码
	- ```java
	      public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
	          if(root == null){
	              return null;
	          }
	          if(root == p || root == q){
	              return root;
	          }
	          
	          TreeNode left = lowestCommonAncestor(root.left,p,q);
	          TreeNode right= lowestCommonAncestor(root.right,p,q);
	          
	          if(left == null){
	              return right;
	          } else if(right == null){
	              return left;
	          } else {
	              return root;
	          }
	      }
	  
	  // 注释版
	  public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
	     		// 如果树为空，直接返回null
	          if(root == null) return null; 
	      // 如果 p和q中有等于 root的，那么它们的最近公共祖先即为root（一个节点也可以是它自己的祖先）
	          if(root == p || root == q) return root;
	     // 递归遍历左子树，只要在左子树中找到了p或q，则先找到谁就返回谁
	          TreeNode left = lowestCommonAncestor(root.left, p, q); 
	     // 递归遍历右子树，只要在右子树中找到了p或q，则先找到谁就返回谁
	          TreeNode right = lowestCommonAncestor(root.right, p, q); 
	     // 如果在左子树中 p和 q都找不到，则 p和 q一定都在右子树中，
	     // 右子树中先遍历到的那个就是最近公共祖先（一个节点也可以是它自己的祖先）
	          if(left == null) return right; 
	     // 否则，如果 left不为空，在左子树中有找到节点（p或q），这时候要再判断一下右子树中的情况，
	     如果在右子树中，p和q都找不到，则 p和q一定都在左子树中，左子树中先遍历到的那个就是最近公共祖先
	       （一个节点也可以是它自己的祖先）
	          else if(right == null) return left;
	     //否则，当 left和 right均不为空时，说明 p、q节点分别在 root异侧, 最近公共祖先即为 root
	          else return root; 
	      }
	  ```