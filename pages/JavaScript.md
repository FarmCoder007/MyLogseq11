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
	- ## 3、var：是最常见的声明变量的关键字。
		- 它没有其他两个关键字的种种限制。这是因为它是传统上在 JavaScript 声明变量的唯一方法。使用 var 声明的变量在它所声明的整个函数都是可见的。
		- ```js
		  var a;
		  var name = "simon";
		  ```
		- 一个使用  var 声明变量的语句块的例子：
		- ```js
		  // myVarVariable 在这里 *能* 被引用
		  
		  for (var myVarVariable = 0; myVarVariable < 5; myVarVariable++) {
		    // myVarVariable 整个函数中都能被引用
		  }
		  
		  // myVarVariable 在这里 *能* 被引用
		  ```
	- JavaScript 与其他语言的（如 Java）的重要区别是在 JavaScript 中语句块（blocks）是没有作用域的，只有函数有作用域。因此如果在一个复合语句中（如 if 控制结构中）使用 var 声明一个变量，那么它的作用域是整个函数（复合语句在函数中）。 但是从 ECMAScript Edition 6 开始将有所不同的， let 和 const 关键字允许你创建块作用域的变量。