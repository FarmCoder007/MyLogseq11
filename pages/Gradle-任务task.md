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
- # 四、给task配置group和描述
  collapsed:: true
	- group可以将任务分组，在AS右侧工具栏gradle的task列表中能够清晰直观的看到group
	- ```groovy
	  //group可以将任务分组，在右侧的task列表中能够清晰直观的看到group
	  task taskDev1{
	      group "dev"
	      description "项目描述"
	      println "method1"
	  }
	  task('taskDev2'){
	      group "dev"
	      println "method2"
	  }
	  
	  task taskTest{
	      group "test"
	      println "method1"
	  }
	  
	  ```
- # 五、doFirst/doLast体验
  collapsed:: true
	- 一个Task包含若干Action。所以，Task有doFirst和doLast两个函数，用于添加需要最先执行的Action和需要和需要最后执行的Action。Action就是一个闭包。
	- ```groovy
	  //在task闭包体内声明
	  task taskDev1{
	      //doFirst：task执行时，最开始的操作
	      doFirst{
	          println "first exec"
	      }
	      //doLast：task执行时，最后的操作
	      doLast{
	          println "last exec"
	      }
	      group "dev"
	      println "method1"
	  }
	  
	  //也可以这样写
	  task myTask {
	      println "config myTask"
	  }
	  myTask.doFirst {
	      println "before execute myTask"
	  }
	  myTask.doLast {
	      println "after execute myTask"
	  }
	  
	  
	  ```
- # 六、任务详细使用
	- ![image.png](../assets/image_1664521928091_0.png)
	- ## 6-1、带参任务
		- ``` 
		  // 定义一个名字为paramTask的task，属于it235分组，并且依赖myTask1和myTask2两个task。
		  task myTask1{
		  }
		  task myTask2{
		  }
		  
		  project.task('paramTask', group: "it235", description: "我自己的Task", dependsOn: ["myTask1", "myTask2"] ).doLast {
		      println "execute paramTask"
		  }
		  
		  ```
-