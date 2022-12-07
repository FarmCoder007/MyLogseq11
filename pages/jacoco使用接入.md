- # 一、AS已经内置了jacoco
	- 只需要开启插件即可
	- ```
	  plugins {
	  	 id 'com.android.application'
	       id 'kotlin-android'
	       id 'jacoco'
	  }
	  
	  jacoco {
	      // 指定版本
	      toolVersion = "0.8.5"
	  }
	  ```
- # 二、开启覆盖率开关
	- ```
	       buildTypes {
	           release {
	               minifyEnabled false
	               // 关闭测试覆盖率
	               testCoverageEnabled = false 
	               proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
	           }
	           debug {
	               // 打开测试覆盖率
	               testCoverageEnabled = true
	           }
	      }
	  
	  ```