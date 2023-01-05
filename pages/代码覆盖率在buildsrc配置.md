- 单元测试代码覆盖率jacoco[[代码覆盖率]]
- gradle已支持jacoco覆盖率配置。只需新增gradle插件如下
- # 方式一、jacoco.gradle
  collapsed:: true
	- ```groovy
	  apply plugin: 'jacoco'
	  
	  
	  android {
	      buildTypes {
	          debug {
	              /**打开覆盖率统计开关**/
	              testCoverageEnabled = true
	          }
	      }
	  }
	  
	  jacoco {
	      toolVersion = "0.8.7"
	  }
	  
	  def coverageSourceDirs = [
	          '../demo-sample/src/main/java'
	  ]
	  
	  // TODO testDebugUnitTest 改成变种的方式获取
	  // type : 代表jacocoTestReport  这个task对应的类 为 JacocoReport
	  // dependsOn： testDebugUnitTest 为 调用jacocoTestReport时在这个task之前先执行testDebugUnitTest
	  task jacocoTestReport(type: JacocoReport, dependsOn: "testDebugUnitTest") {
	      group = "Reporting"
	      description = "Generate Jacoco coverage reports"
	      
	      // 配置过滤
	      classDirectories.from = fileTree(
	              dir: '../demo-sample/build/intermediates/javac/debug/classes',
	              excludes: ['**/R.class',
	                         '**/R$*.class',
	                         '**/*$ViewInjector*.*',
	                         '**/BuildConfig.*',
	                         '**/Manifest*.*']
	      )
	  
	      additionalSourceDirs.from = files(coverageSourceDirs)
	      sourceDirectories.from = files(coverageSourceDirs)
	      executionData.from = files('../demo-sample/build/jacoco/testDebugUnitTest.exec')
	  
	      /**
	       *  是否生成报告
	       */
	      reports {
	          xml.enabled = true
	          html.enabled = true
	      }
	  
	  
	  
	  }
	  ```
- # 方式二、在buildsrc的自定义插件里。初始化jacocoReport task
  collapsed:: true
	- ```kotlin
	  class MetaXAppPlugin : Plugin<Project> {
	      private var apiProject: Project? = null
	      private var libProject: Project? = null
	  
	      private var metaXUnitTest: Task? = null
	      private var jacocoReportTask:JacocoReport?=null
	      override fun apply(project: Project) {
	            project.afterEvaluate { project ->
	              afterEvaluateInit(project, android)
	          }
	      }
	      private fun afterEvaluateInit(project: Project, android: AppExtension?) {
	        initJacocoReportTask(project,android)
	        // 配置jacocoReportTask 变体依赖 testdebugUnitTask  testReleaseUnitTask
	        android?.applicationVariants?.all { variant ->
	              if (!isIntegratedAppModule) {
	                  try {
	                      val testUnitTask = AGPCompat.getTestUnitTest(project, variant.name)
	  //                    metaXUnitTest?.dependsOn(testUnitTask)
	                      jacocoReportTask?.dependsOn(testUnitTask)
	                      orderAssemble(project, variant)
	                  } catch (e: Exception) {
	                      // ignore.
	                  }
	              }
	          }
	      }
	      
	          /**
	       *  初始化jacoco配置
	       */
	      private fun initJacocoReportTask(project: Project,android: AppExtension?){
	          android?.buildTypes?.all {
	              it.isTestCoverageEnabled = true
	          }
	          val jacocoOptions:JacocoOptions? = android?.jacoco
	          jacocoOptions?.version = "0.8.7"
	  
	          jacocoReportTask = project.tasks.create(
	              "jacocoReportTask", JacocoReport::class.java
	          )
	          jacocoReportTask?.group = "metax"
	          jacocoReportTask?.description = "Generate Jacoco coverage reports"
	  
	          val patternSet: PatternFilterable = PatternSet()
	          patternSet.exclude("**/R\$*.class","**/*\$ViewInjector*.*","**/BuildConfig.*",
	              "**/Manifest*.*")
	          jacocoReportTask?.classDirectories?.setFrom(project.fileTree("../demo-sample/build/intermediates/javac/debug/classes").matching(patternSet))
	          
	        jacocoReportTask?.additionalSourceDirs?.setFrom("../demo-sample/src/main/java")
	          jacocoReportTask?.sourceDirectories?.setFrom("../demo-sample/src/main/java")
	          jacocoReportTask?.executionData?.setFrom("../demo-sample/build/jacoco/testDebugUnitTest.exec")
	          jacocoReportTask?.reports?.html?.isEnabled = true
	          jacocoReportTask?.reports?.xml?.isEnabled = true
	      }
	  }
	  ```
- # 三、代码方式配置gradle中路径过滤的代码写法
  collapsed:: true
	- ```kotlin
	   // gradle 中的写法
	   lassDirectories.from = fileTree(
	              dir: '../demo-sample/build/intermediates/javac/debug/classes',
	              excludes: ['**/R.class',
	                         '**/R$*.class',
	                         '**/*$ViewInjector*.*',
	                         '**/BuildConfig.*',
	                         '**/Manifest*.*']
	      )
	      
	  // buildSrc的 代码写法    
	  val patternSet: PatternFilterable = PatternSet()
	          patternSet.exclude("**/R\$*.class","**/*\$ViewInjector*.*","**/BuildConfig.*",
	              "**/Manifest*.*")
	          jacocoReportTask?.classDirectories?.setFrom(project.fileTree("../demo-sample/build/intermediates/javac/debug/classes").matching(patternSet))
	  ```