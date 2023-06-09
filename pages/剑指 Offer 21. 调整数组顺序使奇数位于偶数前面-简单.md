title:: 剑指 Offer 21. 调整数组顺序使奇数位于偶数前面-简单

- # 题目
	- 输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数在数组的前半部分，所有偶数在数组的后半部分。
	- ## 示例
		- ```java
		  输入：nums = [1,2,3,4]
		  输出：[1,3,2,4] 
		  注：[3,1,2,4] 也是正确的答案之一。
		  ```
	- ## 思路：
		- 1、新建数组，左边存奇数，右边存偶数，遍历完 也存完了
		- 2、双指针存左右
		- ```java
		  class Solution {
		      public int[] exchange(int[] nums) {
		          int n = nums.length;
		          int[] res = new int[n];
		          // 双指针存左右
		          int left = 0, right = n - 1;
		          for (int num : nums) {
		              if (num % 2 == 1) {
		                  res[left++] = num;
		              } else {
		                  res[right--] = num;
		              }
		          }
		          return res;
		      }
		  }
		  ```
		- ## 复杂度分析
			- 时间复杂度：O(n)，其中 n 为数组 nums 的长度。只需遍历nums 一次。
			- 空间复杂度：O(1)。结果不计入空间复杂度。
		-