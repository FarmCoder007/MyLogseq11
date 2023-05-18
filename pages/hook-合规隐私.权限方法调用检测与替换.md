title:: hook-合规隐私/权限方法调用检测与替换

- # 1、插件功能结构
  collapsed:: true
	- 框架
	  collapsed:: true
		- ![image.png](../assets/image_1684412281788_0.png)
	- 支持扫描隐私权限方法，替换隐私权限方法等功能。
	- ## 1. 在跟项目的build.gradle文件中添加插件
	  collapsed:: true
		- ```
		  buildscript {
		      repositories {
		         maven { url "http://artifactory.58corp.com:8081/artifactory/android-public" }
		          maven { url "http://artifactory.58corp.com:8081/artifactory/android-local/" }
		        
		      }
		  
		      dependencies {
		          classpath 'com.coofee.rewrite:rewrite:1.0.3'
		      }
		  }
		  ```
	- ## 2. 在Application项目中添加配置
	  collapsed:: true
		- ```
		  apply plugin: 'com.coofee.rewrite'
		  
		  rewrite {
		  
		      scanPermissionMethodCaller {
		         // ...
		      }
		  
		      replaceMethod {
		          // ...
		      }
		  }
		  ```
	- ## 3.执行
	  collapsed:: true
		- 注意执行插件前，需要先clean，否会因为缓存使用旧结果。
		- ```
		  $ ./gradlew clean
		  $ ./gradlew :app:assembleDebug
		  ```
- # 2、例子
	- ## 1、获取android framework方法权限集
		- 首先需要保证已通过sdk manager安装对应compileSdkVersion版本的android源代码**，如下图所示：
		  collapsed:: true
			- ![image.png](../assets/image_1684412673589_0.png)
		- 然后通过执行collectAndroidPermissionMethod任务获取android framework中需要权限的方法集：
		  collapsed:: true
			- ```
			  
			  ```