title:: 剑指 Offer 50. 第一个只出现一次的字符-简单

- ## 题目
	- 在字符串 s 中找出第一个只出现一次的字符。如果没有，返回一个单空格。 s 只包含小写字母。
	- ## 示例1
		- ```java
		  输入：s = "abaccdeff"
		  输出：'b'
		  ```
	- ## 示例2
		- ```java
		  输入：s = "" 
		  输出：' '
		  ```
	- **限制：**
		- `0 <= s 的长度 <= 50000`
- ## 思路
	- 关于出现个数的，都可以用hash表来处理
	- ## 方法一：hash表
		- ```java
		  class Solution {
		      public char firstUniqChar(String s) {
		          Map<Character, Integer> frequency = new HashMap<Character, Integer>();
		          // 第一次遍历 统计个数
		          for (int i = 0; i < s.length(); ++i) {
		              char ch = s.charAt(i);
		              frequency.put(ch, frequency.getOrDefault(ch, 0) + 1);
		          }
		          // 第二次查找为1的
		          for (int i = 0; i < s.length(); ++i) {
		              if (frequency.get(s.charAt(i)) == 1) {
		                  return s.charAt(i);
		              }
		          }
		          return ' ';
		      }
		  }
		  ```
		- 复杂度
			- 时间 O(N)
			-
	- ## 方法二：hash表映射索引
	- ## 方法三：队列
	-