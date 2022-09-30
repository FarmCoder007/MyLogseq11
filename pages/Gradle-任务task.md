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
	- ```groovy
	  public interface TaskContainer extends TaskCollection<Task>, PolymorphicDomainObjectContainer<Task>{
	      
	  }
	  
	  ```
	- ## 3-1、重要的方法
		- ### 3-1-1、用于定位task：
			- findByPath：如果没找到会返回null
				- ```groovy
				  ```