- # 一、gradle默认工程目录
  collapsed:: true
	- ```groovy
	  src/main/
	      src/main/java/
	      src/main/resources/
	  
	  src/test/
	      src/test/java/
	      src/test/jresources/
	  
	  ```
- # 二、改变源代码目录和资源目录
  collapsed:: true
	- ## 更改目录srcDirs
	  collapsed:: true
		- // 这样，gradle就只在src/java下搜源代码，而不是在src/main/java下
		  sourceSets {
		      main {
		          java {
		              srcDirs = ['src/java']
		          }
		          resources {
		              srcDirs = ['src/resources']
		          }
		      }
		  }
	- ## 新增目录 srcDir
	  collapsed:: true
		- ```groovy
		  // 只是想添加额外的目录，而不想覆盖原来的目录，则像下面这样：
		  sourceSets {
		      main {
		          java {
		              srcDir 'thirdParty/src/main/java'
		          }
		      }
		  }
		  // 此时，gradle就会同时在src/main/java和thirdParty/src/main/java两个目录下都进行源代码搜索
		  ```
	- ## 排除目录 exclude
	  collapsed:: true
		- 执行build之后，你在jar或war包的classes中不会见到排除掉的目录
		- ```groovy
		  sourceSets {
		  	main{
		  		java {
		  			exclude 'com/it235/first/ct/**'
		  		}
		  		resources {
		  			exclude 'cert/**'
		  		}
		  	}
		  }
		  
		  我的目录结构
		  first-gradle
		      src
		  		main
		  			java
		  				com.xxx
		  			resources
		  				cert
		  					test.cert
		  				application.properties
		  			
		  
		  ```
	- ## 指定源码输出目录
	  collapsed:: true
		- ```groovy
		  sourceSets {
		  	main {
		          //指定main的源码输出位置
		  		output.resourcesDir = output.classesDir = 'WebContent/WEB-INF/classes/'
		          //源码目录
		  		java.srcDir('src')
		          //资源文件目录
		  		resources.srcDir('src')
		  	}
		  }
		  
		  ```
- # 三、使用场景
	- ## 什么情况下我们会添加一个新的`SourceSets`？
	  collapsed:: true
		- 原始的src/test目录我们一般存放单元测试，但是对于集成测试部分不应存放在src/test目录下，因为这些测试有它们自己的dependencies, classpaths, 以及其他的资源，所以这些测试应该是单独分开的，而不应该和单元测试混为一谈。这样会将2个测试工作进行混淆，也是架构师们不希望看到的局面，所以我们需要重新定义一个用于集成测试的代码环境。但是，集成测试和验收测试也是这个项目的一部分，因此我们也不能为集成测试或验收测试单独建立一个项目。它们也不是子项目，所以用subproject也不合适。
	- ## 新建一个集成测试或者验收测试的SourceSets，把相关的测试资源管理起来。
		- ### 1、在src目录下新建一个intTest作为集成测试的环境，与src/main、src/test保持一致，格式如下
		  collapsed:: true
			- ```groovy
			  first-gradle
			    src
			      intTest  -- 存放集成测试的源码
			  	  java
			  	  resources
			  	main     -- 存放应用源码
			  	  java
			  	  resources
			  	test     -- 存放单元测试源码
			  	  java
			  	  resources
			  
			  ```
		- ### 2、为他们创建一个新的sourceSets
		- ### 3、将所需的依赖项添加到该sourcesSets配置中
		- ### 4、为该源集配置编译和运行时类路径
		  collapsed:: true
			- ```groovy
			  configurations {
			      //intTestImplementation 扩展至implementation，说明它具备生产源码的各项依赖
			  	intTestImplementation.extendsFrom implementation
			  	intTestRuntimeOnly.extendsFrom runtimeOnly
			      
			      //如果集成测试需要单元测试的依赖可以采用如下做法
			      //intTestImplementation.extendsFrom testImplementation
			  }
			  
			  dependencies {
			  	implementation 'org.springframework.boot:spring-boot-starter-web'
			  	testImplementation 'org.springframework.boot:spring-boot-starter-test'
			  	
			      intTestImplementation 'org.springframework.boot:spring-boot-starter-test'
			      
			  }
			  
			  sourceSets {
			      //这里因为我们的intTest目录与闭包命名一致，所以不使用srcDir添加也能自动识别
			  	intTest {
			          //将正式项目的源码编译和运行时路径给到intTest，记住是+=，而不是覆盖
			          compileClasspath += sourceSets.main.output
			          runtimeClasspath += sourceSets.main.output
			  	}
			  }
			  
			  ```
		- ### 5、创建一个任务来运行集成测试
		  collapsed:: true
			- ```groovy
			  //任务类型是Test
			  task integrationTest(type: Test) {
			  	description = 'Runs integration tests.'
			      //这里指定后，该任务将会在IDE右侧的verification分组里面
			  	group = 'verification'
			  
			      //配置已编译的测试类的位置
			  	testClassesDirs = sourceSets.intTest.output.classesDirs
			      //运行我们的集成测试时使用的类路径。
			  	classpath = sourceSets.intTest.runtimeClasspath
			      
			      //声明在test单元测试完成后自动执行集成测试（非必须）
			  	shouldRunAfter test
			      //mustRunAfter表示必须在test任务执行完成之后执行
			  }
			  
			  //定义一个检查任务，让集成测试在检查任务之前运行，如果集成任务构建失败，检查任务也会失败（非必须）
			  check.dependsOn integrationTest
			  
			  ```
		- ### 6、最后注意定义test单元测试引擎
		  collapsed:: true
			- ```groovy
			  tasks.withType(Test) {
			  	useJUnitPlatform()
			  }
			  
			  //以下写法不生效，暂未找到原因
			  test {
			      useJUnitPlatform()
			  }
			  
			  ```
		- ### 7、完整的配置如下
			- ```groovy
			  ```