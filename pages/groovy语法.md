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
	- ## 2-2