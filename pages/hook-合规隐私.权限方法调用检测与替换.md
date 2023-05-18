title:: hook-合规隐私/权限方法调用检测与替换

- # 1、插件功能结构
  collapsed:: true
	- 框架
	  collapsed:: true
		- ![image.png](../assets/image_1684412281788_0.png)
	- 支持扫描隐私权限方法，替换隐私权限方法等功能。
	- ## 1. 在跟项目的build.gradle文件中添加插件
	  collapsed:: true
		- ```
		  buildscript {
		      repositories {
		         maven { url "http://artifactory.58corp.com:8081/artifactory/android-public" }
		          maven { url "http://artifactory.58corp.com:8081/artifactory/android-local/" }
		        
		      }
		  
		      dependencies {
		          classpath 'com.coofee.rewrite:rewrite:1.0.3'
		      }
		  }
		  ```
	- ## 2. 在Application项目中添加配置
	  collapsed:: true
		- ```
		  apply plugin: 'com.coofee.rewrite'
		  
		  rewrite {
		  
		      scanPermissionMethodCaller {
		         // ...
		      }
		  
		      replaceMethod {
		          // ...
		      }
		  }
		  ```
	- ## 3.执行
	  collapsed:: true
		- 注意执行插件前，需要先clean，否会因为缓存使用旧结果。
		- ```
		  $ ./gradlew clean
		  $ ./gradlew :app:assembleDebug
		  ```
- # 2、例子
	- ## 1、获取android framework方法权限集
	  collapsed:: true
		- 首先需要保证已通过sdk manager安装对应compileSdkVersion版本的android源代码**，如下图所示：
		  collapsed:: true
			- ![image.png](../assets/image_1684412673589_0.png)
		- 然后通过执行collectAndroidPermissionMethod任务获取android framework中需要权限的方法集：
		  collapsed:: true
			- ```
			  
			  ```
		- 查看android_framework_class_method_permission.json文件的内容格式大致如下：
		  collapsed:: true
			- ```
			  {
			    "android.telephony.TelephonyManager#getDeviceId": [
			      "android.Manifest.permission.READ_PRIVILEGED_PHONE_STATE"
			    ],
			    "android.telephony.TelephonyManager#getImei": [
			      "android.Manifest.permission.READ_PRIVILEGED_PHONE_STATE"
			    ],
			    "android.telephony.TelephonyManager#getMeid": [
			      "android.Manifest.permission.READ_PRIVILEGED_PHONE_STATE"
			    ]
			  }
			  ```
		-
	- ## 2. scanPermissionMethodCaller（扫描隐私API方法调用）
		- 配置scanPermissionMethodCaller如下所示:
		  collapsed:: true
			- 注意：对于使用ContentResolver的获取联系人/短信等需要权限的行为，暂时不支持通过scanPermissionMethodCaller统计。
			  可以参考ShadowContentResolver.java和replace_method的配置，
			  使用replaceMethod进行运行时拦截，统计权限获取情况。