title:: 剑指 Offer 58 - I. 翻转单词顺序-简单

- ## 题目
  collapsed:: true
	- 输入一个英文句子，翻转句子中单词的顺序，但单词内字符的顺序不变。为简单起见，标点符号和普通字母一样处理。例如输入字符串"I am a student. "，则输出"student. a am I"。
	- ## 示例1
		- ```java
		  输入: "the sky is blue"
		  输出: "blue is sky the"
		  ```
	- ## 示例2
		- ```java
		  输入: "  hello world!  "
		  输出: "world! hello"
		  解释: 输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
		  ```
	- ## 示例3
		- ```java
		  输入: "a good   example"
		  输出: "example good a"
		  解释: 如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。
		  ```
	- **说明：**
		- 无空格字符构成一个单词。
		- 输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
		- 如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。
- ## 思路
	- ## 方式一：双指针
		- ## 分析
			- 倒序遍历（因为反转字符 所以倒序）字符串 s，记录单词左右索引边界 i , j ；
			- 每确定一个单词的边界，则将其添加至单词列表 res ；
			- 最终，将单词列表拼接为字符串，并返回即可。
		- ## 复杂度分析：
			- O(N)
		- 代码
			- ```java
			  class Solution {
			      public String reverseWords(String s) {
			          s = s.trim(); // 删除首尾空格
			          // 倒序遍历
			          int j = s.length() - 1, i = j;
			          StringBuilder res = new StringBuilder();
			          while(i >= 0) {
			              // 搜索首个空格，i走到空格处
			              while(i >= 0 && s.charAt(i) != ' ') i--;
			              // sp 拼接单词
			              res.append(s.substring(i + 1, j + 1) + " "); // 添加单词
			              // 跳过单词间空格
			              while(i >= 0 && s.charAt(i) == ' ') i--; // 跳过单词间空格
			              j = i; // j 指向下个单词的尾字符
			          }
			          return res.toString().trim(); // 转化为字符串并返回
			      }
			  }
			  ```
	- ## 方式二：分割 + 倒序
		- Java ： 以空格为分割符完成字符串分割后，若两单词间有 x>1 个空格，则在单词列表 strs 中，此两单词间会多出 x−1个 “空单词” （即 "" ）。解决方法：倒序遍历单词列表，并将单词逐个添加至 StringBuilder ，遇到空单词时跳过。
		- ![Picture0.png](https://pic.leetcode-cn.com/9ef4a9ea565bf1c2d9209ca94881a77288f90f222476cfd44c418fa3f2d2d7c1-Picture0.png)
		- ## 复杂度
			- O(N)
		- ## 代码
			- ```java
			  class Solution {
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
			  }
			  ```