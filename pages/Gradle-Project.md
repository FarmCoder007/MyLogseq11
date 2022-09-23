- # 一、简介
	- 在gradle中，每一个build.gradle文件对应一个Project实例，我们在build.gradle中编写的内容，就相当于Project实例的属性或方法。
	- 构建初始化期间，Gradle实例化的步骤如下
	  collapsed:: true
		- 给整个构建创建一个Settings实例，一个Settings实例就是一个settings.gradle文件
		- 针对Settings实例的配置，按配置层次扫描并解析配置在settings.gradle中的project。（其中settings中最为重要的属性是include）
		- 针对每个project对应的build.gradle进行初始，并创建Project实例（这里加载project的顺序是按照前面的配置层次进行，即广度扫描加载，这样可以保证父级模块加载完后，子模块才会被加载）
	- 一个完整的project由以下几个对象组成（实际上只由属性和方法组成）
	  collapsed:: true
		- ![image.png](../assets/image_1663923991768_0.png){:height 477, :width 634}
- # 二、属性使用
	- ## 我们可以在build.gradle文件中任意使用gradle提供的属性或方法，如下：
	  collapsed:: true
		- ```groovy
		  //输出当前项目名称
		  println(project.name)
		  
		  //一般在当前build.gradle中使用时，默认会省略project
		  println(name)
		  
		  //输出project中名字为name的属性
		  println(project.property("name"))
		  
		  //指定默认执行的task，即./gradlew不指定task时会执行该task
		  defaultTasks('yourTask')
		  
		  ```
	- ## 属性定义：
		- ### 内置属性可以直接赋值，无需声明
			-