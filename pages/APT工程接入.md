- # 一、工程创建
	- ## 1、创建annotation模块->一定是java library[存放注解]
		- File -> New Module -> Java library
		- ### 1-1、新建注解-@interface 注解类需要用这个来标识
			- ```
			  @Retention(RetentionPolicy.CLASS) //表示编译时的注解 build时编译出代码
			  @Target(ElementType.TYPE) //表示作用在变量上
			  public @interface AutoGenerationAnnotation {
			      // 注解参数
			      String routerPath();
			  }
			  ```
		- ### 1-2、使用注解
			- ```
			  // routerPath传入参数
			  @AutoGenerationAnnotation(routerPath = "INnnServicePath")
			  public interface INnnService {
			  
			      /**
			       * 调起登陆页
			       */
			      public void launch(Context context,int hhe);
			  
			      /**
			       * 执行退出操作
			       */
			      public void logoutAccount(Context context,String qq,boolean ll);
			  }
			  ```
	- ## 2、创建processor模块->一定是java library[解析处理注解]
		- ### 2-1、注解模块gradle依赖配置
			-
			- plugins {
			    id 'java-library'
			    id 'kotlin'
			    id 'kotlin-kapt'
			  }
			- java {
			    sourceCompatibility= JavaVersion.VERSION_1_8
			    targetCompatibility = JavaVersion.VERSION_1_8
			  }
			- dependencies {
			    implementation project(path: ':annotation-lib')
			    //auto-service是Google开源的一个库，可以方便快捷的帮助我们进行组件化开发，避免手动写
			    implementation 'com.google.auto.service:auto-service:1.0-rc6'
			    // kotlin写的 注解处理器 用 kapt
			    kapt'com.google.auto.service:auto-service:1.0-rc6'
			    // java 写的用
			  //    annotationProcessor 'com.google.auto.service:auto-service:1.0-rc6'
			    // 引入javapoet【apt生成java文件时需添加】
			    implementation "com.squareup:javapoet:1.11.1"
			    // 引入 kotlinpoet 【apt生成 kt文件时需要添加】
			    implementation "com.squareup:kotlinpoet:1.11.0"
			  }
		-
	-
	-
-