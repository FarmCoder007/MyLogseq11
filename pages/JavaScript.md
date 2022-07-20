- [官方教程](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript)
- # 一、变量
	- 在 JavaScript 中声明一个新变量的方法是使用关键字 let 、const 和 var：
	- ## 1、let：let 语句声明一个块级作用域的本地变量，并且可选的将其初始化为一个值。
		- ```js
		  let a;
		  let name = 'Simon';
		  ```
		- 下面是使用  let 声明变量作用域的例子：
		- ```js
		  // myLetVariable 在这里 *不能* 被引用
		  
		  for (let myLetVariable = 0; myLetVariable < 5; myLetVariable++) {
		    // myLetVariable 只能在这里引用
		  }
		  
		  // myLetVariable 在这里 *不能* 被引用
		  ```
	- ## 2、const：允许声明一个不可变的常量。这个常量在定义域内总是可见的
		- ```js
		  const Pi = 3.14; // 设置 Pi 的值
		  Pi = 1; // 将会抛出一个错误因为你改变了一个常量的值。
		  ```
	- ## 3、var ：是最常见的声明变量的关键字。
		-
-