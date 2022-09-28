- # 方式一：“maven-publish” 插件
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