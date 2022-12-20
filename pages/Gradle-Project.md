- # 一、简介
  collapsed:: true
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
  collapsed:: true
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
		- ### 1、内置属性可以直接赋值，无需声明
		  collapsed:: true
			- ```groovy
			  group = 'com.it235'
			  version = '1.0.0'
			  
			  ```
		- ### 2、自定义属性可以使用groovy语法，也可以与java语法结合
		  collapsed:: true
			- ```groovy
			  //groovy定义属性
			  def pname = "projectName:" + project.name
			  
			  //java类型接收
			  String pname = "projectName:" + project.name
			  
			  ```
		- ### 3、使用ext名命空间来扩展属性，定义后可以在project、task、subproject中读取和更新
		  collapsed:: true
			- ```groovy
			  ext.prop1 = "it235"
			  ext.prop2 = "君哥聊编程"
			  
			  ```
		- ### 4、使用案例
		  collapsed:: true
			- ```groovy
			  //REPOSITORY_HOME 和 REPOSITORY_URL
			  apply plugin: 'maven'
			  ext {
			    REPOSITORY_HOME = 'http://maven.aliyun.com'
			    REPOSITORY_URL =  REPOSITORY_HOME + "/nexus/content/groups/public"
			  }
			  repositories {
			      // 使用 mavenCentral()时，将远程的仓库替换为自己搭建的仓库
			      maven { 
			          url REPOSITORY_URL 
			      }
			  }
			  
			  uploadArchives {
			      repositories {
			          mavenDeployer {
			              snapshotRepository(url: REPOSITORY_HOME + "/nexus/content/repositories/snapshots/") {
			                  authentication(userName: 'xxx', password: 'xxx')
			              }
			              repository(url: REPOSITORY_HOME + "/nexus/content/repositories/releases/") {
			                  authentication(userName: 'xxx', password: 'xxx')
			              }
			          }
			      }
			  }
			  
			  
			  ```
		- ### 5、属性作用域
			- 读写属性时，Project 会按照下面范围的顺序进行查找的，在某个范围找到属性后就会返回该属性。如果没有找到，会抛出异常。
			- Project 对象自身。这个范围里的属性包含 Project 实现类中定义有 getters 和 setters 方法的所有属性。比如：project.getName() 方法就对应了 name 属性。至于这些属性的可读写性取决于它们是否定义 getters 或者 setters 方法。
			- Project 的ext属性 ( extra ) 。每个 Project 都会维护一个额外属性的映射，它可以包含任意的名称 -> 值对。定义后，此作用域的属性是可读写的。比如：project.ext.prop1 = 'it235' 。
			- 通过插件被添加到 Project 中的扩展属性 ( extensions ) 。每个扩展名都可以作为只读属性使用，其名称与扩展名相同。比如：project.android.compileSdkVersion 。
			- 通过插件添加到 Project 中的约定属性 ( convention ) 。插件可以通过 Project 的 Convention 对象向 Project 中添加属性和方法。此范围的属性的可读可写性取决于约束对象。
			- Project 中 Tasks 。可以使用 Task 的名称作为属性名称来访问task。此范围的属性是只读的。
			- ext的属性和约定属性从项目的父级继承，递归到根项目。此范围的属性是只读的。
		- ### 6、常用的project属性
		  collapsed:: true
			- ![image.png](../assets/image_1663925060278_0.png)
			-
		-
- # 三、方法
  collapsed:: true
	- ## 1、方法作用域
	  collapsed:: true
		- Project 对象自身
		- build.gradle 脚本文件
		- 通过插件添加到 Project 中的扩展 ( extensions ) 。每个扩展都可以当做参数是闭包或 Action 的方法。
		- 插件添加到项目中的约定方法 ( convention ) 。插件可以通过项目的 Convention 对象向项目中添加属性和方法。
		- 项目中的 Tasks 。每个 Task 都会添加一个方法，方法名是任务名，参数是单个闭包或者 Action 。该方法使用提供的闭包为相关任务调用 Task.configure( groovy.lang.Closure ) 方法。
	- ## 2、常用的Project方法
		- ![image.png](../assets/image_1663926685739_0.png)
		- ![image.png](../assets/image_1663926699086_0.png){:height 281, :width 716}
		-
		- ### 常用方法示例：
			- 1、buildscript{}：配置当前gradle脚本自身需要使用的构建信息或依赖
			  collapsed:: true
				- 假设要执行一项指令./gradlew buildImage，构建docker镜像，而Gradle官方自身没有，则需要依赖到maven库下载或需要调用第三方插件，虽然这里是调用的task，但是task背后所依赖的插件是需要提前定义在buildscript中的，我们需要在buildscript{}中指定docker的依赖即可。
				- ```groovy
				  apply plugin: 'idea'
				  apply plugin: 'java'
				  apply plugin: "maven"
				  apply plugin: "war"
				  apply plugin: "com.bmuschko.docker-remote-api"
				  apply plugin: "org.springframework.boot"
				  
				  buildscript {
				      repositories {
				          mavenLocal()
				          maven {
				              url "https://maven.aliyun.com/repository/public"
				          }
				  		mavenCentral()
				      }
				      dependencies {
				          classpath "com.bmuschko:gradle-docker-plugin:3.3.4"
				          classpath "org.springframework.boot:spring-boot-gradle-plugin:2.6.5"
				      }
				  }
				  
				  ```
			- 2、[[gradle-project-configurations依赖配置]]
			- 3、repositories{}：仓库配置
			  collapsed:: true
				- 通过 repositories{} 可以配置maven，ivy，local仓库。这样子，在dependencies{}声明的依赖就可以通过repositories{}中指定的仓库查询到具体的JAR资源。
				- ```groovy
				  repositories {
				  	mavenLocal()
				  	mavenCentral()
				  	maven {
				  		// Name is optional. If not set url property is used
				  		name = 'Main Maven repository'
				  		url = 'https://maven.aliyun.com/repository/central'
				  	}
				  
				  	//有权限控制的仓库
				  	maven() {
				  		credentials {
				  			username = 'username'
				  			password = 'password'
				  		}
				  		url = 'https://maven.aliyun.com/repository/central'
				  	}
				  
				  	//本地仓库
				  	repositories {
				  		flatDir(dir: '../lib', name: 'libs directory')
				  		flatDir {
				  			dirs '../project-files', '/volumes/shared-libs'
				  			name = 'All dependency directories'
				  		}
				  	}
				  }
				  
				  ```
				-
			- 4、[[gradle-project-dependencies依赖]]
			- 5、allprojects{}：配置此项目及其每个子项目所需要的依赖。一般在多模块项目场景下我们会把公共的部分配置在根项目的allprojects中。
			  collapsed:: true
				- ```groovy
				  apply plugin: 'java'
				  apply plugin: "maven"
				  apply plugin: "war"
				  apply plugin: "com.bmuschko.docker-remote-api"
				  apply plugin: "org.springframework.boot"
				  
				  buildscript {
				      repositories {
				          mavenLocal()
				          maven {
				              url "https://maven.aliyun.com/repository/public"
				          }
				  		mavenCentral()
				      }
				      dependencies {
				          classpath "com.bmuschko:gradle-docker-plugin:3.3.4"
				          classpath "org.springframework.boot:spring-boot-gradle-plugin:2.6.5"
				      }
				  }
				  
				  allprojects {
				      apply plugin: "idea"
				  
				      repositories {
				          mavenLocal()
				          maven {url "https://maven.aliyun.com/repository/public"}
				          mavenCentral()
				      }
				  
				      // 依赖管理器
				      dependencyManagement {
				          imports {
				              mavenBom 'org.springframework.cloud:spring-cloud-dependencies:Hoxton.SR12'
				              mavenBom 'com.alibaba.cloud:spring-cloud-alibaba-dependencies:2.2.6.RELEASE'
				          }
				      }
				  
				      // ...各种配置...
				      
				      //依赖
				      dependencies {
				          implementation("org.springframework.boot:spring-boot-starter") {
				              exclude module: "tomcat-embed-el"
				          }
				          implementation("org.springframework.boot:spring-boot-starter-web")
				          api 'com.squareup.retrofit2:retrofit:2.4.0'
				          //仅编译
				          compileOnly 'org.projectlombok:lombok:1.18.16'
				          annotationProcessor 'org.projectlombok:lombok:1.18.16'
				          //仅测试编译
				          testCompileOnly 'org.projectlombok:lombok:1.18.16'
				          testAnnotationProcessor 'org.projectlombok:lombok:1.18.16'
				  
				         // ...依赖超级多的 jar 包，包括公司的包和三方包....
				         implementation "xxx.xxx.xxx"
				         ...
				      }
				  }
				  
				  ```
			- 6、subprojects{}：子模块配置
			- 7、[[gradle-sourceSets{}：配置源码信息]]
			- 8、[[gradle-artifacts{}：配置需要交付的产品组件信息]]
			-
- # 四、Metax-project工具类
  collapsed:: true
	- ```
	  object AGPCompat {
	  
	      fun isMetaXApiLibrary(name: String?): Boolean {
	          return name?.endsWith("-metax-api") ?: false
	      }
	  
	      fun isMetaXApiLibrary(project: Project): Boolean {
	          return isMetaXApiLibrary(PropertiesUtils.getArtifactId(project))
	      }
	  
	      fun isMetaXImplLibrary(name: String?): Boolean {
	          return name?.endsWith("-metax-lib") ?: false
	      }
	  
	      fun isApiModule(project: Project): Boolean {
	          return project.name.endsWith("-api")
	      }
	  
	      fun isLibModule(project: Project): Boolean {
	          return project.name.endsWith("-lib")
	      }
	  
	      fun isApplication(project: Project): Boolean {
	          return project.plugins.hasPlugin("com.android.application")
	      }
	  
	      fun isLibrary(project: Project): Boolean {
	          return project.plugins.hasPlugin("com.android.library")
	      }
	  
	      fun isAndroidModule(project: Project): Boolean {
	          return project.hasProperty("android")
	      }
	  
	      fun isMetaXNormalMode(project: Project): Boolean {
	          val metaXMode = PropertiesUtils.getMetaXMode(project)
	          return TextUtils.isEmpty(metaXMode) || metaXMode == MetaXConstants.MetaXModeNormal
	      }
	  
	      fun isMetaXOnlyLibMode(project: Project): Boolean {
	          val metaXMode = PropertiesUtils.getMetaXMode(project)
	          return metaXMode == MetaXConstants.MetaXModeOnlyLib
	      }
	  
	      fun getCleanTask(project: Project): Task? {
	          val cleanTaskName = "clean"
	          return project.tasks.findByName(cleanTaskName)
	      }
	  
	      fun getAssembleTask(project: Project, variantName: String): Task? {
	          val assembleTaskName = "assemble${variantName.capitalize()}"
	          return project.tasks.findByName(assembleTaskName)
	      }
	  
	      fun getInstallTask(project: Project, variantName: String): Task? {
	          val assembleTaskName = "install${variantName.capitalize()}"
	          return project.tasks.findByName(assembleTaskName)
	      }
	  
	      fun getPreBuildTask(project: Project): Task? {
	          val assembleTaskName = "preBuild"
	          return project.tasks.findByName(assembleTaskName)
	      }
	  
	      fun getDependenciesTask(project: Project): Task? {
	          val assembleTaskName = "dependencies"
	          return project.tasks.findByName(assembleTaskName)
	      }
	  
	      fun getBuildAarPath(project: Project, variantName: String): String {
	          return "${project.projectDir}${File.separator}build${File.separator}outputs" +
	                  "${File.separator}aar${File.separator}${project.name}-$variantName.aar"
	      }
	  
	      fun getApiProject(project: Project): Project? {
	          val projects = project.rootProject.allprojects
	          var apiProject: Project? = null
	          projects.forEach {
	              if (isApiModule(it)) {
	                  apiProject = it
	              }
	          }
	          return apiProject
	      }
	  
	      fun getLibProject(project: Project): Project? {
	          val projects = project.rootProject.allprojects
	          var libProject: Project? = null
	          projects.forEach {
	              if (isLibModule(it)) {
	                  libProject = it
	              }
	          }
	          return libProject
	      }
	  }
	  ```