title:: 剑指 Offer 05. 替换空格-简单

- ## 数据结构-字符串
- ## 题目
	- 请实现一个函数，把字符串 `s` 中的每个空格替换成"%20"。
	- ### **示例 1：**
		- ```java
		  输入：s = "We are happy."
		  输出："We%20are%20happy."
		  ```
	- ### 限制：
		- `0 <= s 的长度 <= 10000`
	- ## 解题思路
		- java中字符串是不可变的
		- 遍历字符用StringBuilder拼接，遇到空格则用%20拼接
	- ## 代码
		- ```java
		  class Solution {
		      public String replaceSpace(String s) {
		          StringBuilder res = new StringBuilder();
		          for(Character c : s.toCharArray())
		          {
		              if(c == ' ') res.append("%20");
		              else res.append(c);
		          }
		          return res.toString();
		      }
		  }
		  ```