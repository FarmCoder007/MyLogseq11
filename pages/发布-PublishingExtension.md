- [官方文档](https://docs.gradle.org/current/dsl/org.gradle.api.publish.PublishingExtension.html)
- 例子：
  collapsed:: true
	- ```groovy
	  plugins {
	      ...
	      id 'maven-publish'
	  	...
	  }
	  
	  publishing {
	      publications {
	          //打springboot-jar包
	          uesJar(MavenPublication) {
	              artifact bootJar
	              from components.java
	          }
	  
	          //可单独执行publishDbchangelogPublicationToUesRepository 上传dbchangelog压缩包
	          dbchangelog(MavenPublication) {
	              artifactId 'ues-db-changelog'
	              artifact dbChangelogZip
	          }
	      }
	  
	      repositories {
	          maven {
	              //忽略https
	              allowInsecureProtocol true
	              name = 'ues'
	              def releasesRepoUrl = "http://192.168.35.xx:8081/nexus/content/repositories/releases/"
	              def snapshotsRepoUrl = "http://192.168.35.xx:8081/nexus/content/repositories/snapshots/"
	              url = version.endsWith('SNAPSHOT') ? snapshotsRepoUrl : releasesRepoUrl
	              credentials {
	                  username 'admin'
	                  password 'admin123'
	              }
	          }
	      }
	  }
	  
	  ```
- # 一、简介PublishingExtension发布扩展
- # 二、生成pom标识值
	- ## 标识值
		- groupId - Project.getGroup() 组名
		  artifactId - Project.getName() 产物名
		  version - Project.getVersion() 发布版本
	- ## 自定义发布标识
		- ```groovy
		  publishing {
		          publications {
		              maven(MavenPublication) {
		                  groupId 'org.gradle.sample'
		                  artifactId 'project1-sample'
		                  version '1.1'
		  
		                  from components.java
		              }
		          }
		      }
		  ```
- # 三、2个容器
  collapsed:: true
	- ## publications：配置此项目的发布。
		- ### 方法细节：void publications(Action<? super PublicationContainer> configure)
		- ```groovy
		  plugins { 
		      id 'maven-publish' 
		  } 
		  
		  publishing { 
		    publications { 
		      // myPublicationName发布名字
		      myPublicationName(MavenPublication) { // 在此处配置发布
		      } 
		    } 
		  }
		  ```
	- ## repositories：配置要发布到的可能存储库的容器。
		- ### void repositories(Action<? super RepositoryHandler> configure)
			- ```groovy
			  plugins {
			      ...
			      id 'maven-publish'
			  	...
			  }
			  
			  publishing {
			      // 发布仓库配置
			      repositories {
			          maven {
			              //忽略https
			              allowInsecureProtocol true
			              name = 'ues'
			              def releasesRepoUrl = "http://192.168.35.xx:8081/nexus/content/repositories/releases/"
			              def snapshotsRepoUrl = "http://192.168.35.xx:8081/nexus/content/repositories/snapshots/"
			              url = version.endsWith('SNAPSHOT') ? snapshotsRepoUrl : releasesRepoUrl
			              credentials {
			                  username 'admin'
			                  password 'admin123'
			              }
			          }
			      }
			  }
			  
			  ```
-
-