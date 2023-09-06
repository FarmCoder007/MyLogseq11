title:: 剑指 Offer 61. 扑克牌中的顺子-简单

- ## [题目](https://leetcode.cn/problems/bu-ke-pai-zhong-de-shun-zi-lcof/solutions/212071/mian-shi-ti-61-bu-ke-pai-zhong-de-shun-zi-ji-he-se/)
  collapsed:: true
	- 从**若干副扑克牌**中随机抽 `5` 张牌，判断是不是一个顺子，即这5张牌是不是连续的。2～10为数字本身，A为1，J为11，Q为12，K为13，而大、小王为 0 ，可以看成任意数字。A 不能视为 14。
	- ## 示例
		- ```java
		  输入: [1,2,3,4,5]
		  输出: True
		  ```
	- ## 示例
		- ```java
		  输入: [0,0,1,2,5]
		  输出: True
		  ```
	- **限制：**
		- 数组长度为 5
		- 数组的数取值为 [0, 13] .
- ## [题解](https://leetcode.cn/problems/bu-ke-pai-zhong-de-shun-zi-lcof/solutions/212071/mian-shi-ti-61-bu-ke-pai-zhong-de-shun-zi-ji-he-se/)
	- ## 方法二hashset排序
	  id:: 64895c43-9025-4a4d-a3b2-82c52ae63d8d
		- 1、hashset去重
		- 2、找数组中最大值 最小值，差小于5就可以
		- ```java
		  class Solution {
		      public boolean isStraight(int[] nums) {
		          // 1、hashset 顺子去重用
		          Set<Integer> repeat = new HashSet<>();
		          int max = 0, min = 14;
		          for(int num : nums) {
		              if(num == 0) continue; // 2、跳过大小王
		              max = Math.max(max, num); // 3、找到数组里最大牌
		              min = Math.min(min, num); // 4、找到数组里最小牌
		              if(repeat.contains(num)) return false; // 若有重复，提前返回 false
		              repeat.add(num); // 添加此牌至 Set
		          }
		          return max - min < 5; // 最大牌 - 最小牌 < 5 则可构成顺子
		      }
		  }
		  ```
	- ##### 方法一、排序
	  collapsed:: true
		- ```java
		  class Solution {
		      public boolean isStraight(int[] nums) {
		          int joker = 0;
		          Arrays.sort(nums); // 数组排序
		          for(int i = 0; i < 4; i++) {
		              if(nums[i] == 0) joker++; // 统计大小王数量
		              else if(nums[i] == nums[i + 1]) return false; // 若有重复，提前返回 false
		          }
		          return nums[4] - nums[joker] < 5; // 最大牌 - 最小牌 < 5 则可构成顺子
		      }
		  }
		  ```