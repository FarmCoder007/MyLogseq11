title:: 剑指 Offer 21. 调整数组顺序使奇数位于偶数前面-简单

- # [题目](https://leetcode.cn/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)
	- 输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数在数组的前半部分，所有偶数在数组的后半部分。
	- ## 示例
		- ```java
		  输入：nums = [1,2,3,4]
		  输出：[1,3,2,4] 
		  注：[3,1,2,4] 也是正确的答案之一。
		  ```
	- ## [[#red]]==**思路：**==
		- 1、定义新数组
		- 2、定义数组左右两端 双指针
		- 3、遍历数组一遍
		- 4、%2取余 遍历数组奇数存左，否则存右
		- ```java
		  class Solution {
		      public int[] exchange(int[] nums) {
		          int n = nums.length;
		          // 1、定义新数组
		          int[] res = new int[n];
		          // 2、定义数组左右两端 双指针
		          int left = 0,right = n-1;
		          for(int num:nums){
		              // 3、%2取余 遍历数组奇数存左
		              if(num%2 ==1){
		                  res[left++] = num;
		              }else{
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