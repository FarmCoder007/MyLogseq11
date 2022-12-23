- 单元测试代码覆盖率jacoco[[代码覆盖率]]
- gradle已支持jacoco覆盖率配置。只需新增gradle插件如下
- # 一、jacoco.gradle
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
	  // dependsOn： testDebugUnitTest 为 调用jacocoTestReport时在这个task
	  task jacocoTestReport(type: JacocoReport, dependsOn: "testDebugUnitTest") {
	      group = "Reporting"
	      description = "Generate Jacoco coverage reports"
	  
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