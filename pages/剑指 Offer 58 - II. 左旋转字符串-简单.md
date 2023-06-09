title:: 剑指 Offer 58 - II. 左旋转字符串-简单

- ## 题目
	- 字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部。请定义一个函数实现字符串左旋转操作的功能。比如，输入字符串"abcdefg"和数字2，该函数将返回左旋转两位得到的结果"cdefgab"。
	- ## 示例1
		- ```java
		  输入: s = "abcdefg", k = 2
		  输出: "cdefgab"
		  ```
	- ## 示例2
		- ```java
		  输入: s = "lrloseumgh", k = 6
		  输出: "umghlrlose"
		  ```
- ## 思路一 subString
  collapsed:: true
	- ![Picture1.png](https://pic.leetcode-cn.com/9c60c78876bc1edfd432364fe35a9d13d86e4cbb5a72eaa70bcd911585b1572b-Picture1.png)
	- ```java
	  class Solution {
	      public String reverseLeftWords(String s, int n) {
	          return s.substring(n, s.length()) + s.substring(0, n);
	      }
	  }
	  ```
- ## 思路二、StringBulder
	- ```java
	  class Solution {
	      public String reverseLeftWords(String s, int n) {
	          StringBuilder res = new StringBuilder();
	          for(int i = n; i < s.length(); i++)
	              res.append(s.charAt(i));
	          for(int i = 0; i < n; i++)
	              res.append(s.charAt(i));
	          return res.toString();
	      }
	  }
	  ```
- ## 思路三、字符串遍历拼接
	- ```java
	  class Solution {
	      public String reverseLeftWords(String s, int n) {
	          String res = "";
	          for(int i = n; i < s.length(); i++)
	              res += s.charAt(i);
	          for(int i = 0; i < n; i++)
	              res += s.charAt(i);
	          return res;
	      }
	  }
	  ```
- ## 复杂度都是O(n)