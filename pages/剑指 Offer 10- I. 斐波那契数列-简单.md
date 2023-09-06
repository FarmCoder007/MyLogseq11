title:: 剑指 Offer 10- I. 斐波那契数列-简单

- # [题目](https://leetcode.cn/problems/fei-bo-na-qi-shu-lie-lcof/)
	- 写一个函数，输入 `n` ，求斐波那契（Fibonacci）数列的第 `n` 项（即 `F(N)`）。斐波那契数列的定义如下：
	- >```
	  F(0) = 0,   F(1) = 1
	  F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
	  ```
	- 斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出。
	- 答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。
	- ## 示例1
		- ```java
		  输入：n = 2
		  输出：1
		  ```
	- ## 示例2：
		- ```java
		  输入：n = 5
		  输出：5
		  ```
	- **提示：**
		- `0 <= n <= 100`
		-
- # 思路
	- 动态规划法，优化O(1)
	- ![动态规划.gif](../assets/动态规划_1686220899791_0.gif)
	- ```java
	  F(0) = 0
	  F(1) = 1
	  F(2) = F(0) + F(1) = 1
	  F(3) = F(2) + F(1) = 2
	  
	  class Solution {
	      public int fib(int n) {
	          int a = 0, b = 1, sum;
	          for (int i = 0; i < n; i++) {
	              sum = (a + b) % 1000000007;
	              a = b;
	              b = sum;
	          }
	          return a;
	      }
	  }
	  ```