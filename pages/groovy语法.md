- # 一、简介
	- Groovy是一种基于JVM的动态语言，语法和java相似，最终也是要编译.class在JVM上运行。
- # 二、语法
	- ## 2-1、标识符
	  collapsed:: true
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
	  collapsed:: true
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
	  collapsed:: true
		- ### 运算符
		  collapsed:: true
			- 算术运算符
			  关系运算符
			  逻辑运算符
			  位运算符
			  赋值运算符
		- ### if else  switch
		  collapsed:: true
			- ```groovy
			  // if条件语句
			  def age = 28
			  if (age >= 30) {
			      println("${age}岁，不小了，该找女票了")
			  } else {
			      println("还小呢，继续玩,才${age}岁")
			  }
			  
			  //if-else-if
			  def age = 28
			  if (age >= 30) {
			      println("${age}岁，不小了，该找女票了")
			  } else if (age < 28 && age > 22) {
			      println("还小呢，继续玩,才${age}岁")
			  }else {
			      println("${age}尴尬的年纪")
			  }
			  
			  //swtich switch支持集合、支持闭包、支持正则
			  
			  def x = 1.23
			  def result = ""
			  switch ( x ) {
			      case "foo":
			          result = "found foo"
			          // lets fall through
			      case "bar":
			          result += "bar"
			      case [4, 5, 6, 'inList']:
			          result = "list"
			          break
			      case 12..30:
			          result = "range"
			          break
			      case Integer:
			          result = "integer"
			          break
			      case Number:
			          result = "number"
			          break
			      case ~/fo*/: // toString() representation of x matches the pattern?
			          result = "foo regex"
			          break
			      case { it < 0 }: // or { x < 0 }
			          result = "negative"
			          break
			      default:
			          result = "default"
			  }
			  
			  ```
		- ### 循环
		  collapsed:: true
			- ```groovy
			  //0 1 2 3 4
			  for(i = 0; i < 5; i++){
			      print i + ' '
			  }
			  
			  //0 1 2 3 4 5
			  for(i in 0..5){
			      print i + ' '
			  }
			  //0 1 2 3 4
			  for(i in 0..<5){
			      print i + ' '
			  }
			  
			  //0 1 2 3 4 
			  5.times{
			      print it + ' '
			  }
			  
			  //5 6 7 8 9 10 
			  5.upto(10){
			      print it + ' '
			  }
			  
			  //5 4 3 2
			  5.downto(2){
			      print it + ' '
			  }
			  
			  //5 8 11 14
			  5.step(17,3){
			      print it + ' '
			  }
			  
			  //5 3 1 -1 -3
			  5.step(-4,-2){
			      print it + ' '
			  }
			  
			  
			  ```
	- ## 2-5、JavaBean
	  collapsed:: true
		- Groovy会为类中每个没有可见性修饰符的字段生成getter/setter方法，我们访问这个字段其实是调用它的getter/setter方法。
		- 同时只要写了 getter/setter 方法，也一样可以作为属性进行访问。
		- 对于没有定义的属性，如果只写了 getter 方法，那不可以修改该属性的值，如例子中的 age 属性
		- ```groovy
		  class Person{
		      def name
		      private def country
		    
		      def setNation(nation){
		          this.country = nation
		      }
		      def getNation(){
		          return country
		      }
		  	public int getAge(){24}
		      
		      public String getInfo(){
		  		"$country-$name-$age"
		  	}
		  }
		  
		  def person = new Person()
		  //访问属性
		  person.name = '君哥'
		  println person.name
		  
		  //像字段一样访问这个以get/set开头的方法
		  person.nation = "china"
		  println person.nation
		  
		  //访问的是info的get方法
		  println person.info
		  
		  ```
	- ## 2-6、集合
	  collapsed:: true
		- List
		  collapsed:: true
			- 对于 List 来说，Groovy 提供下标索引的方式进行访问，值得注意的是，除了普通的下标索引，还有负下标索引和范围索引的方式
			- ```groovy
			  def listTest{
			  	def nameList = ["it235","君哥","Groovy","Java","Kotlin","js","Gradle"]
			  	println nameList[0] // 正数第一个
			  	println nameList[1] // 正数第二个
			  	println nameList[-2] // 倒数第二个
			  	println nameList[-1] // 倒数第一个
			  	println nameList[0..2] // 正数第一个到第三个
			  	println nameList[-1..1] // 倒数第一个到正数第二个，倒着打印
			  }
			  
			  ```
		- Map
		  collapsed:: true
			- Map在gradle中用得非常多，如：plugins中、dependencies中，map所有key默认都是string类型的。
			- ```groovy
			  //空Map声明
			  def mapNull = [:];
			  println mapNull.size();//0
			  
			  def map = [
			          iPhone : 'iWebOS',
			          Android: '2.3.3',
			          Nokia  : 'Symbian',
			          Windows: 'WM8'
			  ]
			  
			  println map instanceof HashMap
			  
			  if(map.containsKey('Windows')){
			      println(map.Windows) 
			      println(map['Windows'])
			      println map.get("Windows","default value")
			  }
			  
			  // Print the values
			  map.each{ k, v -> println "${k}:${v}" }
			  
			  for ( e in map ) {
			      print "key:${e.key},value:${e.value}"
			  }
			  
			  ```
	- ## 2-7、闭包groovy.lang.Closure
	  collapsed:: true
		- 介绍：
		  collapsed:: true
			- 闭包是Groovy中非常重要的一个数据类型，是由大括号括起来的一段可执行的代码，可以接受参数也可以有返回值。
			- 闭包语义{ [closureParameters -> ] statements }
				- 其中[closureParameters->]代表参数列表，多参数用逗号分割，用->隔开参数与内容，没有参数可以不写->
			- ```groovy
			  //一段简单的输出代码块
			  {println "Hello Gradle"}
			  
			  //闭包只有一个参数的时候，默认就是 it，无需申明
			  {println it}
			  
			  //即使申明了it也不影响程序
			  {it -> println it}
			  
			  //将上述申明的it换成name，程序依然能读取到
			  {name -> println name}
			  
			  //多个参数且指定参数类型的闭包
			  {String a ,int b -> println "key:$a ,value:$b"}
			  
			  //多个参数未指定类型的闭包
			  {k,v -> println "$k-$v"}
			  
			  ```
		- groovy.lang.Closure API
		  collapsed:: true
			- 每个定义的闭包是一个Closure对象，所以我们可以把一个闭包赋值给一个变量
			- ```groovy
			  //接收简单类型的闭包对象
			  def simpleClosure = {
			      println "Hello Gradle"
			  }
			  
			  //接收带一个参数的闭包对象
			  def itClosure = {
			      it -> println it
			  }
			  
			  //接收带多个参数的闭包对象，且包含返回值
			  def paramClosure = {
			      k,v -> println "$k-$v"
			      return "hello closure"
			  }
			  
			  ```
		- 执行闭包
			- 行闭包对象有两种，一是直接用括号+参数，二是调用call方法
			- 闭包对象(参数)
			- 闭包对象.call(参数)
			- ```groovy
			  simpleClosure()
			  simpleClosure.call()
			  
			  itClosure("默认1个参数的闭包")
			  itClosure.call("默认1个参数的闭包")
			  
			  def resultA = paramClosure("hello" , "world")
			  def resultB = paramClosure.call("hello" , "world")
			  
			  ```
			- 复杂一点的调用
				- ```groovy
				  //闭包体作为参数传入，操作单个属性
				  def simpleCall(closure){
				  	def value = "Hello Groovy"
				      closure(value)
				  }
				  
				  //作为参数传入
				  simpleCall({print it})
				  
				  //去掉()的简写
				  simpleCall{
				      print it
				  }
				  
				  ```
				- ```groovy
				  //map遍历操作
				  def mapEach(closureTemp){
				  	def map = ["君哥","it235.com","Gradle":"Groovy"]
				  	map.each {
				  		closureTemp(it.key,it.value)
				  	}
				  }
				  
				  //调用mapEach方法，mapEach的参数就是闭包
				  mapEach { 
				  	k,v ->
				      	// 闭包不只一个参数的时候，需要把参数一一列出
				  		println "$k-$v"
				  }
				  
				  
				  ```
				- ```groovy
				  // 方法 listEach 只有一个参数，用于接收一个闭包参数
				  def listEach(closureTemp){
				  	def nameList = ["君哥","it235.com","Groovy","Java","Kotlin"]
				  	nameList.each {
				  		closureTemp(it)
				  	}
				  }
				  
				  // listEach 的参数是闭包
				  listEach {
				      // 闭包只有一个参数的时候，默认就是 it
				      println it
				  }
				  
				  ```
				- 代码块可作为参数传递**：代码块就是闭包，通俗来说就是**一段被大括号包裹起来的代码
				- ```groovy
				  // Closure 定义闭包
				  def doSomething(String a, String b, Closure method){
				  	method.call(a,b)
				  }
				  
				  //方式1：初始写法，比较繁琐
				  doSomething("君哥","it235.com",{a,b -> println "$a-$b"})
				  
				  //方式2：Groovy 规定如果方法的最后一个参数是闭包，可以放到方法后面
				  doSomething("君哥","it235.com"){ a,b ->
				      println "$a-$b"
				  }
				  //方式3：括号可以省略，最常见的写法
				  doSomething "君哥","it235.com",{ a,b ->
				      println "$a-$b"
				  }
				  
				  ```
-