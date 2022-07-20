- # [官方教程](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript)
- # 一、变量
  collapsed:: true
	- 在 JavaScript 中声明一个新变量的方法是使用关键字 let 、const 和 var：
	- ## 1、let：let 语句声明一个块级作用域的本地变量，并且可选的将其初始化为一个值。
	  collapsed:: true
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
	  collapsed:: true
		- ```js
		  const Pi = 3.14; // 设置 Pi 的值
		  Pi = 1; // 将会抛出一个错误因为你改变了一个常量的值。
		  ```
	- ## 3、var：是最常见的声明变量的关键字。
	  collapsed:: true
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
- # 二、对象(类似java中的hashMap)
  collapsed:: true
	- JavaScript 中的对象，Object，可以简单理解成“名称 - 值”对（而不是键值对：现在，ES 2015 的映射表（Map），比对象更接近键值对），不难联想 JavaScript 中的对象与下面这些概念类似：
		- Python 中的字典（Dictionary）
		- Perl 和 Ruby 中的散列/哈希（Hash）
		- C/C++ 中的散列表（Hash table）
		- Java 中的散列映射表（HashMap）
		- PHP 中的关联数组（Associative array）
	- “名称”部分是一个 JavaScript 字符串，“值”部分可以是任何 JavaScript 的数据类型——包括对象。这使用户可以根据具体需求，创建出相当复杂的数据结构。
	- ## 创建对象
		- ```js
		  var obj = new Object();
		  或者
		  var obj = {}; // 对象字面量（object literal）”法 也是json的核心语法
		  ```
		- ```js
		  var obj = {
		      name: "Carrot",
		      _for: "Max",//'for' 是保留字之一，使用'_for'代替
		      details: {
		          color: "orange",
		          size: 12
		      }
		  }
		  
		  ```
			- 对象的属性可以通过链式（chain）表示方法进行访问：
			- ```js
			  obj.details.color; // orange
			  obj["details"]["size"]; // 12
			  ```
	- ## 定义对象原型(也就是定义java中的类)
		- ```js
		  function Person(name, age) {
		    this.name = name;
		    this.age = age;
		  }
		  
		  // 定义一个对象
		  var You = new Person('You', 24);
		  // 我们创建了一个新的 Person，名称是 "You"
		  // ("You" 是第一个参数，24 是第二个参数..)
		  
		  完成创建后，对象属性可以通过如下两种方式进行赋值和访问：
		  
		  // 点表示法 (dot notation)
		  obj.name = 'Simon';
		  var name = obj.name;
		  
		  // 括号表示法 (bracket notation)
		  obj['name'] = 'Simon';
		  var name = obj['name'];
		  // can use a variable to define a key
		  var user = prompt('what is your key?')
		  obj[user] = prompt('what is its value?')
		  ```
- # 三、函数
	-