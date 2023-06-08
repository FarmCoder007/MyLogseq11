- ## 题目：翻转“algorithm”字符串
- ## 思路：
	- 1、先将字符串转换字符数组
	- 2、2个指针，1个指向a，1个指向m,进行交换
	- 3、交换完，2个指针向中间移动，直到相遇
	- ![翻转字符串示例.gif](../assets/翻转字符串示例_1685854835657_0.gif)
- ## 思路
	- 1、将字符串转换为字符数组，以便进行字符交换操作。
	- 2、定义两个指针，一个指向字符串的起始位置（称为 left 指针），一个指向字符串的末尾位置（称为 right 指针）。
	- 3、通过交换 left 和 right 指针所指向的字符，实现字符翻转。
	- 4、同时移动 left 指针向右移动一步，移动 right 指针向左移动一步，继续进行字符交换操作。
	- 5、重复步骤 3 和步骤 4，直到 left 指针大于等于 right 指针。
	- 6、将字符数组转换回字符串形式，并返回翻转后的字符串。
- ## 实现：
	- ```java
	   public static String reverseString(String str) {
	          char[] chars = str.toCharArray();
	          int left = 0;
	          int right = chars.length - 1;
	  
	          while (left < right) {
	              // 交换字符
	              char temp = chars[left];
	              chars[left] = chars[right];
	              chars[right] = temp;
	  
	              // 移动指针
	              left++;
	              right--;
	          }
	  
	          return new String(chars);
	      }
	  ```
-