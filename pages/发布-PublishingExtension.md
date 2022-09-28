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
		- Maven 限制了“groupId”和“artifactId”为有限的字符集（[A-Za-z0-9_\\-.]+）
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
- # 四、修改生成的 POM [官方](https://docs.gradle.org/current/dsl/org.gradle.api.publish.maven.MavenPom.html)
  collapsed:: true
	- 从项目信息生成的POM文件可能需要在发布之前进行一些调整。“maven-publish”插件提供了一个钩子以允许这一类的修改。
	- 在这个例子中我们添加了一个用于生成的 POM 的“描述”元素。通过这个钩子，你可以修改 POM 的任何方面的内容。例如，你可以使用生产构建的实际版本号来替换依赖的版本范围。
	- ```groovy
	  publications {
	          // mavenCustom 为发布配置的名字  自定义
	          mavenCustom(MavenPublication) {
	              pom.withXml {
	                  asNode().appendNode('description', 'A demonstration of maven POM customization')
	              }
	          }
	      }
	  ```
- # 五、发布多个模块
	- 有时候从你的 Gradle 构建中发布多个模块会很有用，而不是创建一个单独的 Gradle 子项目。一个例子是为您的library 分别发布一个单独的 API 和它的实现的 jar。使用 Gradle 的话很简单︰