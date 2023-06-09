title:: 剑指 Offer 65. 不用加减乘除做加法-简单

- ## 题目
	- 写一个函数，求两个整数之和，要求在函数体内不得使用 “+”、“-”、“*”、“/” 四则运算符号。
	- ## 示例
		- ```java
		  输入: a = 1, b = 1
		  输出: 2
		  ```
- ## 思路[地址](https://leetcode.cn/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/solutions/210882/mian-shi-ti-65-bu-yong-jia-jian-cheng-chu-zuo-ji-7/)
	- ```java
	  class Solution {
	      public int add(int a, int b) {
	          while(b != 0) { // 当进位为 0 时跳出
	              int c = (a & b) << 1;  // c = 进位
	              a ^= b; // a = 非进位和
	              b = c; // b = 进位
	          }
	          return a;
	      }
	  }
	  ```