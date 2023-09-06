title:: 剑指 Offer 39. 数组中出现次数超过一半的数字-简单

- # [题目](https://leetcode.cn/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/)
	- 数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。
	-
	- 你可以假设数组是非空的，并且给定的数组总是存在多数元素。
	- ## 示例
		- ```java
		  输入: [1, 2, 3, 2, 2, 2, 5, 4, 2]
		  输出: 2
		  ```
- # 思路
	- ## 最佳：- 摩尔投票法： 核心理念为 票数正负抵消 。此方法时间和空间复杂度分别为 O(N) 和 O(1) ，为本题的最佳解法。
	  collapsed:: true
		- ![image.png](../assets/image_1693732240555_0.png)
	- 1、认定一个为众数，票数加1，下一个值 ！= 认定众数  则票数减一
	- ```java
	  public int majorityElement(int[] nums) {
	          // 1、初始众数和票数变量
	          int x = 0, votes = 0;
	          for(int num : nums){
	              // 2、票数为0 初始当前值为假定的众数
	              if(votes == 0) x = num;
	              // 3、当前值num 和 众数相等 则票数+1 
	              votes += num == x ? 1 : -1;
	          }
	          // 返回众数
	          return x;
	      }
	  ```
	- ## 其他
	  collapsed:: true
		- ## 方法一：Hash表
		  collapsed:: true
			- 1、hashMap 统计每个出现的次数
			- 2、遍历hashMap 找到大于数组一半的
			- 代码
			  collapsed:: true
				- ```java
				  class Solution {
				      private Map<Integer, Integer> countNums(int[] nums) {
				          Map<Integer, Integer> counts = new HashMap<Integer, Integer>();
				          for (int num : nums) {
				              if (!counts.containsKey(num)) {
				                  counts.put(num, 1);
				              } else {
				                  counts.put(num, counts.get(num) + 1);
				              }
				          }
				          return counts;
				      }
				  
				      public int majorityElement(int[] nums) {
				          Map<Integer, Integer> counts = countNums(nums);
				  
				          Map.Entry<Integer, Integer> majorityEntry = null;
				          for (Map.Entry<Integer, Integer> entry : counts.entrySet()) {
				              if (majorityEntry == null || entry.getValue() > majorityEntry.getValue()) {
				                  majorityEntry = entry;
				              }
				          }
				  
				          return majorityEntry.getKey();
				      }
				  }
				  
				  
				  ```
			- ## 复杂度
				- 都是O(n)
		- ## 方法二：排序
		  collapsed:: true
			- 如果将数组 nums 中的所有元素按照单调递增或单调递减的顺序排序， ![image.png](../assets/image_1686233226974_0.png) 
			  ​
			- 排序后取中位数
			- ## 代码
			  collapsed:: true
				- ```java
				  class Solution {
				      public int majorityElement(int[] nums) {
				          Arrays.sort(nums);
				          return nums[nums.length / 2];
				      }
				  }
				  ```
			- ## 复杂度
				- ![image.png](../assets/image_1686233288166_0.png)
- 1、初始众数和票数变量
- 2、foreach遍历
	- 1、当票数为0 初始当前值为假定的众数
	- 2、当前值num 和 众数相等 则票数+1
- 3、返回众数x