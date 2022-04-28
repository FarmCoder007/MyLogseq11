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
			- ```
			  plugins {
			      // 作为java library 新建时自动添加
			      id 'java-library'
			      // 使用kotlin 编写processor时需要添加如下配置，下面依赖才能使用akpt
			      id 'kotlin'
			      id 'kotlin-kapt'
			  }
			  
			  java {
			      sourceCompatibility= JavaVersion.VERSION_1_8
			      targetCompatibility = JavaVersion.VERSION_1_8
			  }
			  
			  dependencies {
			      // 依赖注解库
			      implementation project(path: ':annotation-lib')
			      //auto-service是Google开源的一个库，可以方便快捷的帮助我们进行组件化开发，避免手动写
			      // AutoService会自动在build/classes输入目录下生成文件META-INF/services/javax.annotation.processing.Processor,省的手动配置
			      implementation 'com.google.auto.service:auto-service:1.0-rc6'
			      
			      
			      // kotlin写的 注解处理器processor 用 kapt才能被识别
			      kapt'com.google.auto.service:auto-service:1.0-rc6'
			      // java 写的processor 用annotationProcessor 才能被auto-service识别
			  //    annotationProcessor 'com.google.auto.service:auto-service:1.0-rc6'
			     
			     
			     // 引入javapoet【apt生成java文件时需添加】
			      implementation "com.squareup:javapoet:1.11.1"
			      // 引入 kotlinpoet 【apt生成 kt文件时需要添加】
			      implementation "com.squareup:kotlinpoet:1.11.0"
			  }
			  ```
		-
	-
	-
-