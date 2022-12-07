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
  collapsed:: true
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
-
-
- # 参考资料
	- [Android 代码覆盖率如何实现]([125049195](https://blog.csdn.net/m0_71524094/article/details/125049195))