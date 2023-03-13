- # 一、简介
  collapsed:: true
	- 插件可以封装一系列任务，例如 编译，测试，打包等
- # 二、二进制插件：
  collapsed:: true
	- 我们可以通过二进制插件的ID来应用
	- 插件 ID 是插件的全局唯一标识符或者名字
	- ## 老版本使用
	  collapsed:: true
		- ```groovy
		  //build.gradle中的顶级语句，如下分别是使用java/idea/war/docker插件
		  apply plugin: 'java'
		  //apply plugin: JavaPlugin 也可以通过指定插件类来应用，与java效果一样
		  apply plugin: 'idea'
		  apply plugin: "war"
		  //声明
		  apply plugin: "com.jfrog.bintray"
		  apply plugin: 'org.akhikhl.gretty'
		  
		  //buildscript 配置插件仓库地址
		  buildscript {
		    repositories {
		      maven {url "https://maven.aliyun.com/repository/public"}
		      maven { url 'https://maven.aliyun.com/repository/jcenter' }
		    }
		    //应用插件
		    dependencies {
		      classpath "com.jfrog.bintray.gradle:gradle-bintray-plugin:1.8.0"
		      classpath 'org.akhikhl.gretty:gretty:+'
		    }
		  }
		  
		  
		  ```
	- ## plugins{} 新写法
	  collapsed:: true
		- ```groovy
		  //build.gradle中的顶级语句，声明和应用放在一起
		  plugins {
		      //核心插件，gradle提供
		      id 'java'
		      id 'eclipse'
		      id 'war'
		      //非核心插件(社区插件)，必须通过id+version的全限定名的方式进行引用
		      id 'com.bmuschko.docker-remote-api' version '6.7.0'
		      //apply false 表示在父项目中声明，但是父项目不会立即加载，可以在子项目中通过ID的方式进行使用
		      id 'com.jfrog.bintray' version '1.8.5' apply false
		  }
		  //注意：plugins暂时不支持第三方插件，如果要使用第三方插件请使用老的写法。同时plugins中不能随意编写其他的语句体
		  
		  ```
	- ## apply false的使用场景
	  collapsed:: true
		- ```groovy
		  //settings.gradle，有3个子项目
		  include 'hello-a'
		  include 'hello-b'
		  include 'goodbye-c'
		  
		  //根项目的build.gradle
		  plugins {
		      //apply false 表示根项目不会解析加载，只进行定义和插件的版本管理
		      id 'com.example.hello' version '1.0.0' apply false
		      id 'com.example.goodbye' version '1.0.0' apply false
		  }
		  
		  //hello-a/build.gradle
		  plugins {
		      id 'com.example.hello'
		  }
		  
		  //hello-b/build.gradle
		  plugins {
		      id 'com.example.hello'
		  }
		  
		  //goodbye-c/build.gradle
		  plugins {
		      id 'com.example.goodbye'
		  }
		  
		  ```
- # 三、脚本插件：如xxx.gradle,脚本模块化基础
  collapsed:: true
	- 脚本插件通常是一个脚本，和一个普通的 build.gradle 文件没什么区别
	- 它是脚本模块化的基础。我们可以把复杂的脚本文件，进行分块，分段整理，拆分成一个个职责分明的脚本插件。
	- ### 使用：
		- ```groovy
		  //使用本地插件
		  apply from: './other.gradle'
		  apply from: this.rootProject.file('other.gradle')
		  
		  //使用网络远程插件
		  apply from: 'https://gitee.com/xxx/xx/raw/master/it235.gradle'
		  
		  ```
- # 四、自定义gradle插件
	- 插件自定义包括三个步骤：建立插件工程、配置参数、发布插件与使用插件。
	- ## 方式一：可以直接在build.gradle中编写插件，
	  collapsed:: true
		- 优点：当前项目能够自动编译和加载，
		- 缺点：该插件在构建脚本之外不可见。
		- ```groovy
		  class GreetingPlugin implements Plugin<Project> {
		      void apply(Project project) {
		          project.task('hello') {
		              doLast {
		                  println 'Hello from the GreetingPlugin'
		              }
		          }
		      }
		  }
		  
		  // Apply the plugin
		  apply plugin: GreetingPlugin
		  
		  //使用 gradle -q hello 执行即可
		  
		  ```
	- ## 方式二：buildSrc project中
		- 您可以将插件的源代码放在以下目录中（不同语言编写用不同的目录）
			- rootProjectDir/buildSrc/src/main/java
			- rootProjectDir/buildSrc/src/main/groovy
			- rootProjectDir/buildSrc/src/main/kotlin
		- Gradle 将负责编译和测试插件，并使其在构建脚本的类路径中可用。该插件对构建使用的每个构建脚本都是可见的。但是，它在构建之外是不可见的，因此您不能在定义它的构建之外重用插件。
	- ## 方式三：独立工程中
	  collapsed:: true
		- 创建一个单独的项目。该项目生成并发布一个 JAR，然后就可以在多个项目中使用该插件，其他开发者也能下载使用。我们一般会在该jar中编写或依赖一些插件，或者将几个相关的任务类捆绑到一起。
		- #