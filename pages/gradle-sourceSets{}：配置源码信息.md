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
	- ## 排除目录
		- ``````