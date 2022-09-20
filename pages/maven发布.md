- [发布aar包到maven仓库](https://blog.csdn.net/wangsen927/article/details/120720932)
- 发布脚本：
  collapsed:: true
	- ```
	  
	  import org.gradle.api.publish.internal.DefaultPublishingExtension
	  import org.gradle.api.publish.maven.internal.artifact.FileBasedMavenArtifact
	  import org.gradle.api.publish.maven.internal.artifact.DefaultMavenArtifactSet
	  import org.gradle.api.publish.maven.internal.publication.DefaultMavenPublication
	  
	  apply plugin: 'maven-publish'
	  
	  task wubaPublish(group:"wuba") << {
	      //定义一个自己的发布aar的任务
	      println "发布aar完成"
	  }
	  
	  publishing {
	      repositories {
	          maven {
	              def releasesRepoUrl = "http://artifactory.58corp.com:8081/artifactory/android-local"
	              def snapshotsRepoUrl = "http://artifactory.58corp.com:8081/artifactory/android-local"
	              url Boolean.parseBoolean(project.isDebug)?snapshotsRepoUrl : releasesRepoUrl
	              credentials {
	                  username MAVEN_USERNAME
	                  password MAVEN_PASSWORD
	              }
	  
	          }
	      }
	      publications {
	          aar(MavenPublication){
	              groupId GROUP_ID
	              artifactId ARTIFACT_ID
	              version "${project.publish_version}"
	              pom.withXml {
	                  if(Boolean.parseBoolean(project.needPom)){
	                      // 拿到 
	                      ConfigurationContainer configuration = project.configurations;
	                      def dependenciesNode = asNode().appendNode('dependencies')
	                      if(configuration.implementation) {
	                          configuration.implementation.allDependencies.each {
	                              println "****************implementation groupId=${it.group}|${it.name}|${it.version}***********"
	                              if (it.group == "com.huawei") {
	                                  genPomXml(dependenciesNode,it,"implementation")
	                              }
	                          }
	                      }
	                  }
	              }
	          }
	  
	      }
	  }
	  
	  def genPomXml(dependenciesNode,dependency,scope){
	      if(dependenciesNode != null && dependency != null && scope != null){
	          if (dependency.group != null && !'unspecified'.equals(dependency.group)&& !'58ClientProject'.equals(dependency.group)
	                  && dependency.name != null && !'unspecified'.equals(dependency.name)
	                  && dependency.version != null && !'unspecified'.equals(dependency.version)) {
	              def dependencyNode = dependenciesNode.appendNode('dependency')
	              dependencyNode.appendNode('groupId', dependency.group)
	              dependencyNode.appendNode('artifactId', dependency.name)
	              dependencyNode.appendNode('version', dependency.version)
	              dependencyNode.appendNode('scope', "${scope}")
	  
	          }
	      }
	  }
	  
	  def ASSEMBLE_FLAVOR = Boolean.parseBoolean(project.isDebug) ? "Debug" : "Release"
	  
	  //配置任务依赖，发布前先完成构建，并指定发布的aar文件
	  afterEvaluate{
	      DefaultPublishingExtension publishing = project.extensions.findByName("publishing")
	      PublicationContainer publications = publishing.publications
	      DefaultMavenPublication aarPublication = publications.findByName("aar")
	  
	      def aarFile = "${project.getProjectDir()}/libs/pay58sdk-release.aar"
	      changeMavenArtifact(aarPublication,aarFile)//修改要发布的aar路径
	      addPublishAARTask()
	  }
	  
	  
	  
	  /**
	   * 配置发布aar的任务依赖
	   * @param project
	   * @return
	   */
	  def addPublishAARTask(){
	      def ASSEMBLE_FLAVOR = Boolean.parseBoolean(project.isDebug)? "Debug" : "Release"
	      println "ASSEMBLE_FLAVOR : ${ASSEMBLE_FLAVOR} ,project.isDebug = ${project.isDebug}"
	      Task publishTask = project.getTasksByName("publish",false).getAt(0)
	      Task assembleTask = project.getTasksByName("assemble${ASSEMBLE_FLAVOR}",false).getAt(0)
	      Task aarTask = project.getTasksByName("wubaPublish",false).getAt(0)
	  
	      aarTask.dependsOn assembleTask
	      aarTask.dependsOn publishTask
	      publishTask.mustRunAfter assembleTask
	  }
	  
	  /**
	   * 动态改变发布文件的地址
	   * @param publication
	   * @param artifactPath
	   * @return
	   */
	  def changeMavenArtifact(publication, artifactPath){
	      DefaultMavenArtifactSet artifactSet = publication.getArtifacts()
	      artifactSet.clear()
	      MavenArtifact artifact = new FileBasedMavenArtifact(new File(artifactPath))
	      artifactSet.add(artifact)
	  }
	  ```
- gradle.properties
	- ```
	  # 发布aar的版本号
	  publish_version=3.10.2
	  GROUP_ID =mis-mobile
	  ARTIFACT_ID = pay58sdk
	  MAVEN_USERNAME = wuxiandeploy
	  MAVEN_PASSWORD = wuxiandeploy
	  isDebug = false
	  needPom = true
	  ```