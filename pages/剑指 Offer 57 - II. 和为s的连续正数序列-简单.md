title:: 剑指 Offer 57 - II. 和为s的连续正数序列-简单

- ## 题目
	- 输入一个正整数 `target` ，输出所有和为 `target` 的连续正整数序列（至少含有两个数）。
	- 序列内的数字由小到大排列，不同序列按照首个数字从小到大排列。
	- ## 示例
		- ```java
		  输入：target = 9
		  输出：[[2,3,4],[4,5]]
		  ```
	- ## 示例2
		- ```java
		  输入：target = 15
		  输出：[[1,2,3,4,5],[4,5,6],[7,8]]
		  ```
- ## 解题思路 [[滑动窗口]]
	- ## 算法流程：
		- 1、初始化： 左边界 i=1 ，右边界 j=2 ，元素和 s=3 ，结果列表 res ；
		- 2、循环：[[#red]]==当 i≥j 时跳出==；
			- 当 s>target 时： 向右移动左边界 i=i+1 ，并更新元素和 s ；
			- 当 s<target时： 向右移动右边界 j=j+1 ，并更新元素和 s ；
			- 当 s=target 时： 记录连续整数序列，并向右移动左边界 i=i+1；
		- 3、返回值： 返回结果列表 res ；
	- ## 复杂度
		- 时间O(N)
		- 空间O(1)
	- 当 target=9 时，以上求解流程如下图所示：
	  collapsed:: true
		- ![滑动窗口求解.png](../assets/滑动窗口求解_1686299629256_0.png)
- ## 代码
	- ```java
	  class Solution {
	      public int[][] findContinuousSequence(int target) {
	          // i j  左右  s求和
	          int i = 1, j = 2, s = 3;
	          // 记录求和的数组
	          List<int[]> res = new ArrayList<>();
	          while(i < j) {
	              // 找到 目标
	              if(s == target) {
	                  // 记录数组
	                  int[] ans = new int[j - i + 1];
	                  for(int k = i; k <= j; k++)
	                      ans[k - i] = k;
	                  res.add(ans);
	              }
	              // 大=目标 右移左侧
	              if(s >= target) {
	                  s -= i;
	                  i++;
	              } else { // 小于目标右移动  右侧指针
	                  j++;
	                  s += j;
	              }
	          }
	          return res.toArray(new int[0][]);
	      }
	  }
	  ```