- # 一、简介
	- 任务是gradle的最小执行单元，一个build.gradle是由一系列的task组成
- # 二、定义task
  collapsed:: true
	- ## 2-1、groovy定义task闭包
	  collapsed:: true
		- ```groovy
		  //直接声明
		  task taskOne{
		      println "method1"
		  }
		  
		  //参数定义名称
		  task('taskTwo'){
		      println "method2"
		  }
		  
		  //使用tasks集合属性，引用了TaskContainer的一个实例对象。我们还可以使用Project.getTasks()
		  tasks.create('taskThree'){
		      println "method3"
		  }
		  
		  //使用with
		  tasks.with(){
		      println "methodContainer"
		  }
		  
		  //使用withType声明
		  tasks.withType(Test){
		      print "method5"
		  }
		  //使用register,register执行的是延迟创建。也就是说只有当task被需要使用的时候才会被创建。
		  tasks.register('taskSix'){
		      println "method6"
		  }
		  
		  //这种写法gradle5.0就已经废弃
		  task deprecatedTask << {
		      println "execute deprecatedTask"
		  }
		  
		  
		  ```
	- ## 2-2、代码编写
- # 三、TaskContainer用来管理一组task集合
  collapsed:: true
	- ```groovy
	  public interface TaskContainer extends TaskCollection<Task>, PolymorphicDomainObjectContainer<Task>{
	      
	  }
	  
	  ```
		-
	- ## 3-1、用于定位task：
	  collapsed:: true
		- findByPath：如果没找到会返回null
			- ```groovy
			  task hello 
			  println tasks.findByPath('hello').path 
			  println tasks.findByPath(':hello').path
			  
			  ```
		- getByPath：getByPath没找到的话会抛出UnknownTaskException
			- ```groovy
			  task hello 
			  println tasks.getByPath('hello').path 
			  println tasks.getByPath(':hello').path
			  
			  ```
	- ## 3-2、直接创建任务—create
	  collapsed:: true
		- ```groovy
		  class CustomTask extends DefaultTask { 
		      final String message final int number  CustomTask(String message, int number) { this.message = message this.number = number } } 
		  
		  
		  ```
	- ## 3-3、延迟创建任务—register
		- register返回了一个TaskProvider，和java多线程中的callable类似，当我们调用Provider.get()获取task值的时候，才会去创建这个task。
	- ## 3-4、替换任务—replace
		- replace的作用就是创建一个新的task，并且替换掉同样名字的老的task。