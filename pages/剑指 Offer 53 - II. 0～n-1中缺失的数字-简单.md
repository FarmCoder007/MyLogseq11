title:: 剑指 Offer 53 - II. 0～n-1中缺失的数字-简单

- # 题目
	- 一个长度为n-1的递增排序数组中的所有数字都是唯一的，并且每个数字都在范围0～n-1之内。在范围0～n-1内的n个数字中有且只有一个数字不在该数组中，请找出这个数字。
	- ## **示例 1:**
		- ```java
		  输入: [0,1,3]
		  输出: 2
		  ```
	- ## 示例2：
		- ```java
		  输入: [0,1,2,3,4,5,6,7,9]
		  输出: 8
		  ```
	-
- ## 方法一：哈希
  collapsed:: true
	- ```java
	  class Solution {
	      public int missingNumber(int[] nums) {
	          Set<Integer> set = new HashSet<Integer>();
	          // 不缺少数字的数量
	          int n = nums.length + 1;
	          // 遍历nums 为 n-1个数  存入 
	          for (int i = 0; i < n - 1; i++) {
	              set.add(nums[i]);
	          }
	          int missing = -1;
	          // 遍历 n 验证缺少值
	          for (int i = 0; i <= n - 1; i++) {
	              if (!set.contains(i)) {
	                  missing = i;
	                  break;
	              }
	          }
	          return missing;
	      }
	  }
	  ```
- ## 方法二：直接遍历
  collapsed:: true
	- 由于数组已经按递增顺序排序，因此可以根据数组中每个下标处的元素是否和下标相等，得到缺失的数字。
	- ```java
	  class Solution {
	      public int missingNumber(int[] nums) {
	          // 不缺少应该是n个
	          int n = nums.length + 1;
	          
	          for (int i = 0; i < n - 1; i++) {
	              if (nums[i] != i) {
	                  return i;
	              }
	          }
	          return n - 1;
	      }
	  }
	  ```
- ## 方法三：高斯求和
  collapsed:: true
	- ![image.png](../assets/image_1686296659382_0.png)
	- ```java
	  class Solution {
	      public int missingNumber(int[] nums) {
	          int n = nums.length + 1;
	          int total = n * (n - 1) / 2;
	          int arrSum = 0;
	          for (int i = 0; i < n - 1; i++) {
	              arrSum += nums[i];
	          }
	          return total - arrSum;
	      }
	  }
	  ```