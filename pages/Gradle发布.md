- # 方式一：“maven-publish” 插件
	- 简介：使用Maven格式发布的功能，是由 “maven-publish” 插件提供的。
	- “publishing” 插件在project上创建了一个名为 “publishing”的 [[发布-PublishingExtension]]类型的扩展。这个扩展提供了两个容器，一个叫publications，一个叫repositories。“maven-publish”插件适用于MavenPublication publications 和 MavenArtifactRepository 仓库。
	- publishing{}：deploy当前项目到仓库
		- ```groovy
		  ```
- # 发布脚本例子：
  collapsed:: true
	- ```groovy
	  import org.gradle.api.publish.internal.DefaultPublishingExtension
	  import org.gradle.api.publish.maven.internal.artifact.FileBasedMavenArtifact
	  import org.gradle.api.publish.maven.internal.artifact.DefaultMavenArtifactSet
	  import org.gradle.api.publish.maven.internal.publication.DefaultMavenPublication
	  
	  apply plugin: 'maven-publish'
	  
	  task wubaPublish(group:"wuba") {
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
	              artifactId project.getName()
	              version "${project.publish_version}"
	              pom.withXml {
	                  if(true){
	                      // 获取工程的依赖配置 将工程中的build.gradle 中的dependencies中的依赖打入pom
	                      ConfigurationContainer configuration = project.configurations
	                      def dependenciesNode = asNode().appendNode('dependencies')
	                      if(configuration.compile) {
	                          configuration.compile.allDependencies.each {
	                              println "****************compile groupId=${it.group}|${it.name}|${it.version}***********"
	                              genPomXml(dependenciesNode,it,"compile")
	                          }
	                      }
	                      if(configuration.implementation) {
	                          configuration.implementation.allDependencies.each {
	                              println "****************implementation groupId=${it.group}|${it.name}|${it.version}***********"
	                              genPomXml(dependenciesNode,it,"implementation")
	                          }
	                      }
	                      if(configuration.api) {
	                          configuration.api.allDependencies.each {
	                              println "****************api groupId=${it.group}|${it.name}|${it.version}***********"
	                              genPomXml(dependenciesNode,it,"api")
	                          }
	                      }
	                  }
	              }
	          }
	  
	      }
	  }
	  
	  /**
	   pom文件
	  * <dependencies>
	  	<dependency>
	  		<groupId>com.huawei</groupId>
	  		<artifactId>petalpaycheckoutsdk</artifactId>
	  		<version>1.0.9.001.3</version>
	  		<scope>implementation</scope>
	  	</dependency>
	  	<dependency>
	  		<groupId>com.huawei</groupId>
	          <artifactId>petalpaycheckoutsdkaidl</artifactId>
	          <version>1.0.9.001.3</version>
	          <scope>implementation</scope>
	  	</dependency>
	  	<dependency>
	          <groupId>com.mikesamuel</groupId>
	          <artifactId>json-sanitizer</artifactId>
	          <version>1.2.2</version>
	          <scope>implementation</scope>
	  	</dependency>
	  </dependencies>
	  *
	  */
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
	  
	  //执行入口、配置任务依赖，发布前先完成构建，并指定发布的aar文件
	  afterEvaluate{
	      // 拿到发布扩展publishing
	      DefaultPublishingExtension publishing = project.extensions.findByName("publishing")
	      PublicationContainer publications = publishing.publications
	      // 获取上边配置的发布配置
	      DefaultMavenPublication aarPublication = publications.findByName("aar")
	      // 拿到发布要构建的aar 
	      def aarFile = "${project.getProjectDir()}/build/outputs/aar/${project.getName()}-${ASSEMBLE_FLAVOR.toLowerCase()}.aar"
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
	      // 获取发布任务 
	      Task publishTask = project.getTasksByName("publish",false).getAt(0)
	      // 构建任务
	      Task assembleTask = project.getTasksByName("assemble${ASSEMBLE_FLAVOR}",false).getAt(0)
	      // aartask为上边配置打印log
	      Task aarTask = project.getTasksByName("wubaPublish",false).getAt(0)
	  
	      aarTask.dependsOn assembleTask
	      aarTask.dependsOn publishTask
	      // 发布任务在 构建任务之后
	      publishTask.mustRunAfter assembleTask
	  }
	  
	  /**
	   * 动态改变发布文件的地址，动态改版发布的aar路径
	   * 作用：动态改版发布的内容。
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
-