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