title:: 剑指 Offer 42. 连续子数组的最大和-简单

- # 题目
	- 输入一个整型数组，数组中的一个或连续多个整数组成一个子数组。求所有子数组的和的最大值。
	- 要求时间复杂度为O(n)。
	- ## **示例1:**
		- ```java
		  输入: nums = [-2,1,-3,4,-1,2,1,-5,4]
		  输出: 6
		  解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。
		  ```
- # 思路-动态规划
  collapsed:: true
	- **[[#red]]==状态定义==**： 设动态规划列表 dp ，==**dp[i] 代表以元素 nums[i] 为结尾的连续子数组最大和**==。
		- 为何定义最大和 dp[i] 中必须包含元素 nums[i] ：
		- 保证 dp[i] 递推到 dp[i+1] 的正确性；如果不包含 nums[i] ，递推时则不满足题目的 连续子数组 要求。
	- ==**转移方程**==： 若 dp[i−1]≤0 ，说明 dp[i−1] 对dp[i] 产生负贡献，即 dp[i−1]+nums[i]还不如 nums[i] 本身大。
		- 当 dp[i−1]>0 时：执行 dp[i]=dp[i−1]+nums[i]
		- 当 dp[i−1]≤0 时：执行 dp[i]=nums[i] ；
	- ==**初始状态**==： dp[0]=nums[0]，即以nums[0] 结尾的连续子数组最大和为 nums[0] 。
	- ==**返回值**==： 返回 dp 列表中的最大值，代表全局最大值。
	- ![Picture1.png](https://pic.leetcode-cn.com/8fec91e89a69d8695be2974de14b74905fcd60393921492bbe0338b0a628fd9a-Picture1.png)
- # 简单描述
  collapsed:: true
	- 1、利用动态规划数组dp[]存储  以nums[i]  为结尾的 最大子数组的和。dp每个元素都是以对应num元素为结尾的最大和
	- 2、再求 dp最大和数组中的  最大值即可
- # 复杂度分析：
  collapsed:: true
	- 时间复杂度 O(N)： 线性遍历数组 nums 即可获得结果，使用 O(N) 时间。
	  空间复杂度 O(1) ： 使用常数大小的额外空间。
- # 降低空间复杂度
  collapsed:: true
	- 由于 dp[i]只与 dp[i−1]和 nums[i]有关系，因此可以将原数组nums 用作 dp 列表，即直接在 nums 上修改即可。
	  由于省去 dp 列表使用的额外空间，因此空间复杂度从O(N) 降至 O(1) 。
- # 代码
	- ```java
	  class Solution {
	      // -2 1 -3 4 -1 2 1 -5 4 
	      public int maxSubArray(int[] nums) {
	          // 0 为第一个记录最大值-2
	          int res = nums[0];
	          for(int i = 1; i < nums.length; i++) {
	              // i = 1 时 nums[1] += 0 因为nums[0] 是负数，叠加会使值变小
	              // i = 2 时 nums[2] += nums[1] = -2 更新该位置值
	              // i = 3    nums[3] += 0 = 4 因为nums[2]是-2
	              // 
	              // 直接在当前列表nums 看是否叠加前一个值（有可能值已经变为了和）  
	              nums[i] += Math.max(nums[i - 1], 0);
	              // 更新完列表值，不断和当前记录的最大值对比
	              res = Math.max(res, nums[i]);
	          }
	          return res;
	      }
	  }
	  ```