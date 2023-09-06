title:: 剑指 Offer 53 - I. 在排序数组中查找数字 I-简单

- ## 题目
	- 统计一个数字在排序数组中出现的次数。
	- ## 示例
		- ```java
		  输入: nums = [5,7,7,8,8,10], target = 8
		  输出: 2
		  ```
	- ## 示例
		- ```java
		  输入: nums = [5,7,7,8,8,10], target = 6
		  输出: 0
		  ```
- # 排序数组：二分查找
	- 排序数组 nums 中的所有数字 target 形成一个窗口，记窗口的 左 / 右边界 索引分别为 left和 right ，分别对应窗口左边 / 右边的首个元素。
	- 本题要求统计数字 target的出现次数，可转化为：使用二分法分别找到 左边界 left 和 右边界 right ([[#red]]==**因为数组是有序的，找个数就是找左边界，右边界)，易得数字 target的数量为 right−left−1 。**==
	- ![Picture1.png](https://pic.leetcode-cn.com/b4521d9ba346cad9e382017d1abd1db2304b4521d4f2d839c32d0ecff17a9c0d-Picture1.png)
	- 复杂的
	  collapsed:: true
		- ```java
		  class Solution {
		      public int search(int[] nums, int target) {
		          // 搜索右边界 right
		          int i = 0, j = nums.length - 1;
		          while(i <= j) {
		              int m = (i + j) / 2;
		              if(nums[m] <= target) i = m + 1;
		              else j = m - 1;
		          }
		          int right = i;
		        
		        
		        
		          // 若数组中无 target ，则提前返回
		          if(j >= 0 && nums[j] != target) return 0;
		          // 搜索左边界 right
		          i = 0; j = nums.length - 1;
		          while(i <= j) {
		              int m = (i + j) / 2;
		              if(nums[m] < target) i = m + 1;
		              else j = m - 1;
		          }
		          int left = j;
		          return right - left - 1;
		      }
		  }
		  ```
- ## 代码
	- 以上代码显得比较臃肿（两轮二分查找代码冗余）。为简化代码，可将**二分查找右边界 right的代码封装至函数 `helper()` 。
	- ```java
	  class Solution {
	      public int search(int[] nums, int target) {
	          // tagget 右边界   -  target-1的右边界  就是target的个数
	          return helper(nums, target) - helper(nums, target - 1);
	      }
	      
	      // 寻找目标右边界
	      int helper(int[] nums, int tar) {
	          // 1、相撞指针 i  j
	          int i = 0, j = nums.length - 1;
	          while(i <= j) {
	              // 2、取中
	              int m = (i + j) / 2;
	              // 3、中数 小于目标  i往右侧
	              if(nums[m] <= tar) i = m + 1;
	              // 4、中数 大于目标   j往左
	              else j = m - 1;     
	          }
	          return i; // num[m] = tar 
	      }
	  }
	  ```
-