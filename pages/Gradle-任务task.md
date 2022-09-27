- # 一、简介
	- 任务是gradle的最小执行单元，一个build.gradle是由一系列的task组成
- # 二、定义task
	- ## 2-1、groovy定义task闭包
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
	- ## 2-2、代码编写的插件