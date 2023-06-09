title:: 剑指 Offer 57. 和为s的两个数字-简单

- ## 题目
	- 输入一个递增排序的数组和一个数字s，在数组中查找两个数，使得它们的和正好是s。如果有多对数字的和等于s，则输出任意一对即可
	- ## 示例1
		- ```java
		  输入：nums = [2,7,11,15], target = 9
		  输出：[2,7] 或者 [7,2]
		  ```
	- ## 示例2
		- ```java
		  输入：nums = [10,26,30,31,47,60], target = 40
		  输出：[10,30] 或者 [30,10]
		  ```
	- **限制：**
		- `1 <= nums.length <= 10^5`
		- `1 <= nums[i] <= 10^6`
- ## 思路
	- 双指针
	- ```java
	  class Solution {
	      public int[] twoSum(int[] nums, int target) {
	          // 一个位于 数组左侧 一个位于右侧
	          int i = 0, j = nums.length - 1;
	          while(i < j) {
	            	// 和
	              int s = nums[i] + nums[j];
	              if(s < target) i++;
	              else if(s > target) j--;
	              else return new int[] { nums[i], nums[j] };
	          }
	          return new int[0];
	      }
	  }
	  ```