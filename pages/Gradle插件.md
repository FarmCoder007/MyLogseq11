- # 一、简介
  collapsed:: true
	- 插件可以封装一系列任务，例如 编译，测试，打包等
- # 二、二进制插件：
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
		- ```groovy
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
-