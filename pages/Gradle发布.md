- # 方式一：“maven-publish” 插件
	- 简介：使用Maven格式发布的功能，是由 “maven-publish” 插件提供的。
	- “publishing” 插件在project上创建了一个名为 “publishing”的 PublishingExtension类型的扩展。这个扩展提供了两个容器，一个叫publications，一个叫repositories。“maven-publish”插件适用于MavenPublication publications 和 MavenArtifactRepository 仓库。
	- publishing{}：deploy当前项目到仓库
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