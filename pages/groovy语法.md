- # 一、简介
	- Groovy是一种基于JVM的动态语言，语法和java相似，最终也是要编译.class在JVM上运行。
- # 二、语法
	- ## 2-1、标识符
		- def 是 Groovy 用来定义标识符的关键字
		- ```groovy
		  // 定义一个字符串变量 domain 它的值为 it235
		  def domain = "it235";
		  
		  // 定义一个方法 buildString，接收两个字符串参数，返回它们拼接的结果
		  def buildString(String a, String b){
		  	println "$a-$b"
		  }
		  
		  // 结尾的分号可要可不要
		  def result = buildString('hello' , 'it235')
		  
		  ```
	- ## 2-2、字符串
	  collapsed:: true
		- Groovy 中提供了三种表示 String 的方法：单引号（’），双引号（"）或三引号（’’’）括起来。
		- 单引号括起来的字符串没有运算能力，表示普通字符串
		- 双引号表示的字符串可以使用取值运算符‘\${}’，而$在单引号只只是表示一个普通的字符
		- 三引号表示的字符串又称为模版字符串，可以跨越多行，它可以保留文本的换行和缩进格式，三引号同样不支持$
		- ```groovy
		  def stringTest() {
		  	def strA = '单引号'
		      def strB = "双引号"
		      def strC = '''
		  ---三引号---
		  '''
		      println strA
		      println strB
		      println strC
		      println '单引号==$strB'
		      println "双引号==$strB"
		      println '''三引号==$strC'''
		  }
		  
		  ```
	- ## 2-3、方法
		- 括号可省略： 调用方法的时候，可以省略括号，如 #2
		  collapsed:: true
			- ```groovy
			  def methodTest(){
			      buildString("Nicholas","hzf") // #1
			      buildString "Nicholas","hzf" // #2
			  }
			  
			  def buildString(String a, String b){
			  	println "$a-$b"
			  }
			  
			  ```
		- return 可不写： 方法中没有写 return 语句，则方法执行过程中的最后一句代码的值作为返回值
		  collapsed:: true
			- ```groovy
			  def methodTest(){
			      def result = chooseSmallNum 1,10
			      println result
			  }
			  
			  def chooseSmallNum(int a, int b){
			  	if (a <= b) {
			  		a
			  	}else {
			  		b
			  	}
			  }
			  
			  ```
		- Groovy支持动态类型推导，使用def来定义变量和方法时可以不指定变量的类型和方法的返回值类型，同时定义方法时参数可以不指定类型
		  collapsed:: true
			- ```groovy
			  def num1 = 1
			  def num2 = 2
			  def result = add(num1, num2)
			  
			  def add(a, b){
			      return a + b
			  }
			  
			  ```
	- ## 2-4、运算符与逻辑控制
		- ### 运算符
		  collapsed:: true
			- 算术运算符
			  关系运算符
			  逻辑运算符
			  位运算符
			  赋值运算符
		- ###f else  swi
			- i