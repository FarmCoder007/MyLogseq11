- 1、Given a 32-bit signed integer,reverse digits of an integer.(给定一个32位带有符号的整数，将其反转)。
	- 例如 输入 123  输出321  例如输入-123 输出-321 例如输入 120 输出21
	- 注意：只能处理32位int范围内的整数，超过范围返回0
- # 剑指offer
	- # [[#red]]==**第一天**==
	  collapsed:: true
		- ## [[数组]]
			- ## 解决问题思路
				- 1、排序数组中的搜索问题，首先想到[[#red]]==**二分查找**==解决2
				- 2、重复数字问题  hashset、原地交换nums[nums[i]] == nums[i]
				- 3、插入排序，快速排序
				- 4、摩尔投票法
			- [[剑指 Offer 53 - II. 0～n-1中缺失的数字-简单]] 二分查找 ==腾讯==
			  collapsed:: true
				- 1、记住有序数组，不缺少肯定nums[i] = i的，这就是二分条件 nums[mid] == mid
				- 2、缺少一个分为左右2块，左侧nums[i] = i 右侧 nums[i] != i
				- 3、找到右侧第一个不等的，就是二分的条件
				- ```java
				  class Solution {
				      public int missingNumber(int[] nums) {
				          int left = 0, right = nums.length - 1;
				          while(left <= right){
				              int mid = (left + right) / 2;
				              // 值和下标相等，说明缺失的位于右侧
				              if(nums[mid] == mid){
				                  left = mid + 1;
				              }else { // 值和下标不等，缺失的位于左侧区间
				                  right = mid - 1;
				              }
				          }
				          return left;
				      }
				  }
				  ```
			- [[剑指 Offer 03. 数组中重复的数字-简单]]值与下标进行交换 ==字节 小米==
			  collapsed:: true
				- 遍历数组 nums ，设索引初始值为 i=0 :
					- 若 nums[i]=i ： 说明此数字已在对应索引位置，无需交换，因此跳过；
					- 若 nums[nums[i]]=nums[i] ： 代表索引nums[i] 处和索引 i 处的元素值都为 nums[i] ，即找到一组重复值，返回此值 nums[i] ；
					- 否则： 交换索引为 i 和 nums[i] 的元素值，将此数字交换至对应索引位置。
				- 若遍历完毕尚未返回，则返回 −1 。
				- ![image.png](../assets/image_1693719666118_0.png)
			- [[剑指 Offer 21. 调整数组顺序使奇数位于偶数前面-简单]]双指针模2遍历 ==字节 美团 小米==
			  collapsed:: true
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
			- [[如何在未排序整数数组中找到最大值和最小值?]] 直接遍历，2个变量存 ==字节==
			- [[一个巨大无序数组中，查找一个不连续的自然数的节点]]  插入排序 ==喜马拉雅==
			- [[删除有序数组中的重复项-简单]] 快慢指针，快指针找不同，慢指针负责存==字节 小米==
			- [[剑指 Offer 40. 最小的k个数-简单]]-[[#red]]==**快速排序 真题**==
			- [[剑指 Offer 39. 数组中出现次数超过一半的数字-简单]]-摩尔投票法
			  collapsed:: true
				- 1、初始众数和票数变量
				- 2、foreach遍历
					- 1、当票数为0 初始当前值为假定的众数
					- 2、当前值num 和 众数相等 则票数+1
				- 3、返回众数x
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
			- [[剑指 Offer 11. 旋转数组的最小数字-简单]] 查找用二分，字节
			- [[剑指 Offer 57. 和为s的两个数字-简单]] 对撞双指针，字节 百度
			- [[剑指 Offer 61. 扑克牌中的顺子-简单]]遍历数组hashset去重  ==字节百度==
			- [[剑指 Offer 53 - I. 在排序数组中查找数字 I-简单]]-二分查找
	- # [[#red]]==第二天==
	  collapsed:: true
		- ## 字符串
		  collapsed:: true
			- [[String算法-翻转字符串]] 碰撞指针交换
			- [[557. 反转字符串中的单词]]
				- 1、按空格分组
				- 2、每个字符串旋转
				- 3、拼接
			- [[剑指 Offer 58 - I. 翻转单词顺序-简单]] split+sb倒序拼接
			  collapsed:: true
				- 1、split+倒序
				- ```java
				  public String reverseWords(String s) {
				         // 删除首尾空格，分割字符串
				          String[] strs = s.trim().split(" ");
				          StringBuilder res = new StringBuilder();
				          // 倒序遍历单词列表
				          for(int i = strs.length - 1; i >= 0; i--) { 
				              if(strs[i].equals("")) continue; // 遇到空单词则跳过
				              res.append(strs[i] + " "); // 将单词拼接至 StringBuilder
				          }
				          return res.toString().trim(); // 转化为字符串，删除尾部空格，并返回
				      }
				  ```
			- [[20.有效括号]]-hashmap+辅助栈
			  collapsed:: true
				- 1、hashmap存储 成对括号
				- 2、遍历字符
				- 3、左括号为map里指定的key，则加入栈
				- 4、否则肯定为右括号。直接从栈弹出key map[key] = 右括号 看是否成对
			- [[剑指 Offer 50. 第一个只出现一次的字符-简单]] hashmap 统计char 个数
				- 1、第一次遍历字符串  统计个数
				- 2、第二次遍历字符串  map 按字符查找个数为1的
			- [[剑指 Offer 58 - II. 左旋转字符串-简单]] sb或者 string遍历拼接
				- 遍历拼接
			- [[125. 验证回文串-简单]] 左右碰撞指针 对比对应字符
				- 1、定义左右碰撞指针
				- 2、从左右分别找到 是字母或者 数字的下标
				- 3、转小写 对比 是否一样，不一样直接return false
				- 4、left  right 向中间移动
			- [[242.有效的字母异位词]] hashmap 存储个数 两个串字符个数抵消
				- 1、foreach s map字符+1
				- 2、foreach t map字符-1
				- 3、foreach map.values  看值是否正负抵消 都为0
			- [[剑指 Offer 05. 替换空格-简单]]  sb拼接
				- 1、遍历字符
				- 2、sb拼接 遇到空格就拼接别的
		- ## 斐波那契数列
		  collapsed:: true
			- [[剑指 Offer 10- I. 斐波那契数列-简单]]
			- [[剑指 Offer 10- II. 青蛙跳台阶问题-简单]]
		- ## 数学/动态规划
		  collapsed:: true
			- [[剑指 Offer 42. 连续子数组的最大和-简单]] 动态规划-  原数组修改值。 判断前一个是否大于0 才叠加
			  collapsed:: true
				- 1、默认0位置为最大值
				- 2、从index = 1开始遍历，看是否可以和前一个值叠加  并赋值
				- 3、对比叠加后的nums[i] 和 max  更新 max
				- ```java
				      public int maxSubArray(int[] nums) {
				          if(nums.length == 0 ){
				              return 0;
				          }
				          // 1、默认0位置为最大值
				          int max = nums[0];
				          // 2、从index = 1 开始遍历
				          for (int i = 1; i < nums.length; i++) {
				              // 1、看前一个是否 可以赋值叠加
				              nums[i] += Math.max(nums[i-1],0);
				              // 2、
				              max = Math.max(nums[i],max);
				          }
				          return max;
				      }
				  ```
	-
	- # [[#red]]==第三天==
	  collapsed:: true
		- ## 链表
			- 1、一个数组插入删除查找和链表的效率对比？如果一个数组要反复插入删除怎么优化降低时间复杂度？==腾讯==
			  collapsed:: true
				- 面试官提示其实就是垃圾回收的算法 原理就是“标记-查找”。
				- 每次删除的时候元素不是真的被删除了，而是先标记，最后统一移动数组元素，减少移动次数）
			- 2、ArrayList查询第一个跟最后一个复杂度一样么？（一样，下标取值）==腾讯==
			  collapsed:: true
				- 那LinkedList查询第一个跟最后一个复杂度一样么？（一样，linkedlist是双向链表 一样的）
			- [[如何得到单链表的长度?-单链表遍历while循环]]360
			  collapsed:: true
				- 获取头节点遍历 同时计数
			-
			-
			- [[206.反转链表（剑指 Offer II 024. 反转链表）-简单]]
			- [[剑指 Offer 06. 从尾到头打印链表-简单]]
			- [[剑指 Offer 22. 链表中倒数第k个节点-简单]]
			- [[剑指 Offer 25. 合并两个排序的链表-简单]]
			- [[剑指 Offer 18. 删除链表的节点-简单]]
			- [[剑指 Offer 52. 两个链表的第一个公共节点-简单]]
			- [[环形链表，判断是否有环-快慢指针]]
			- [[如果链表有环，查找环的起始点]]
			- [[如果链表存在环，如何判断环的长度？]]
		- ## 栈
			- [[剑指 Offer 09. 用两个栈实现队列-简单]]
			- [[剑指 Offer 30. 包含min函数的栈-简单]]
	- # [[#red]]==第四天==
	  collapsed:: true
		- ## [[二叉树]]
			- ## 考点：递归
				- [[剑指 Offer 28. 对称的二叉树-简单]]
			- ## 考点：查找
				- [[剑指 Offer 54. 二叉搜索树的第k大节点-简单]]
					- 二叉搜索树的中序遍历为 **递增序列**
			- ## 考点：层序遍历（广度优先，借助队列）
				- [[剑指 Offer 32 - II. 从上到下打印二叉树 II-简单]]
			- ## 考点：深度(遍历)
				- [[剑指 Offer 55 - I. 二叉树的深度-简单]]
			- [[剑指 Offer 68 - II. 二叉树的最近公共祖先-简单]]
			- [[剑指 Offer 68 - I. 二叉搜索树的最近公共祖先-简单]]
			- [[剑指 Offer 55 - II. 平衡二叉树-简单]]
			- [[剑指 Offer 27. 二叉树的镜像-简单]]
			-
	-
	-
	- # 放弃
	  collapsed:: true
		- ## [[位运算]]
		  collapsed:: true
			- [[剑指 Offer 65. 不用加减乘除做加法-简单]]
			- [[剑指 Offer 15. 二进制中1的个数-简单]]
		- ## 数组
			- ## 放弃
			  collapsed:: true
				- [[剑指 Offer 17. 打印从1到最大的n位数-简单]]
				- [[无序数组中第k个最大元素-中等]] 快速排序-字节腾讯 放弃
				- [[剑指 Offer 57 - II. 和为s的连续正数序列-简单]]
		- ## [[滑动窗口]]
			- [[剑指 Offer 57 - II. 和为s的连续正数序列-简单]]
-