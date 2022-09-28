- [官方文档](https://docs.gradle.org/current/dsl/org.gradle.api.publish.PublishingExtension.html)
- 例子：
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
- # 二、2个容器
	- ## publications：配置此项目的发布。
		- ### 方法细节：void publications(Action<? super PublicationContainer> configure)
		- ```groovy
		  ```
	- ## repositories：配置要发布到的可能存储库的容器。
-