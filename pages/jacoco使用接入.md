- # 一、AS已经内置了jacoco
  collapsed:: true
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
- # 三、jacoco报告覆盖率为0 问题排查
  collapsed:: true
	- 借助jacococli工具，打印exec里的文件，jacococli.jar 已上传到csdn
		- ```
		  java -jar jacococli.jar execinfo /Users/xuwenbin/AndroidStudioProjects/58ComponentProject2/MetaXUtils/demo-sample/build/jacoco/testDebugUnitTest.exec 
		  ```
-
-
- # 原理介绍
  collapsed:: true
	- jacoco即是通过修改class文件的字节码来进行代码覆盖率统计的。即，在原有class字节码中的指定位置插入探针字节码，形成新的字节码指令流。jacoco使用的是ASM字节码框架对字节码进行修改的（关于ASM框架，可以参考这篇文章：https://www.cnblogs.com/liuling/archive/2013/05/25/asm.html）。jacoco的探针实际是一个布尔值，当代码执行到探针位置时，将其置为true，该探针前面的代码会被认为执行过，然后对该部分代码对应的html文件中的css样式进行染色（红色表示未覆盖，绿色表示已覆盖，黄色表示部分覆盖），形成最终的覆盖率报告。
- # 报告分析
  collapsed:: true
	- ![image.png](../assets/image_1672046620315_0.png)
-
- # 参考资料
	- [Android 代码覆盖率如何实现]([125049195](https://blog.csdn.net/m0_71524094/article/details/125049195))
	- [jacoco在android studio 上的配置使用](https://blog.csdn.net/u010663321/article/details/121761825)
	- [Android Jacoco覆盖率统计配置](https://github.com/hanlyjiang/AndroidTestSample/blob/main/docs/Jacoco%E8%A6%86%E7%9B%96%E7%8E%87%E7%BB%9F%E8%AE%A1%E6%96%B9%E6%A1%88.md)
	- [JUnit+Robolectric+JaCoCo实现Android单元测试及覆盖率检查](https://blog.csdn.net/yugong2009/article/details/80462094)
	- [汇总多模块测试结果和多模块jacoco代码覆盖率](https://www.cnblogs.com/xy-ouyang/p/16098978.html)
-