# 一、概念题
	- ## 1、[[with函数多个重载->作用域]]
		- ## 作用域总结：
		  collapsed:: true
			- ### Application作用域
				- 情况
					- [[#red]]==**子线程执行Glide.with(this)**==
					- 或者主线程执行with（Application Context），传入参数为Service Context 或者 Application Context
				- 特点：==**不会搞空白Fragment监听生命周期。**==
			- ### 非Application作用域：
				- 主线程
					- 传入VIew-> 作用域Activity / Fragment
					- 传入Fragment -> 作用域Fragment
					- 传入Activity -> 作用域Activity
				- 特点：会添加空白Fragment 监听生命周期
	- ## 2、用到的设计模式
		- 1、单例
		  collapsed:: true
			- ```java
			    @NonNull
			    public static Glide get(@NonNull Context context) {
			      if (glide == null) {
			        GeneratedAppGlideModule annotationGeneratedModule =
			            getAnnotationGeneratedGlideModules(context.getApplicationContext());
			        synchronized (Glide.class) {
			          if (glide == null) {
			            checkAndInitializeGlide(context, annotationGeneratedModule);
			          }
			        }
			      }
			  
			      return glide;
			    }
			  ```
		- 2、工厂设计模式
		  collapsed:: true
			- ![image.png](../assets/image_1691897351242_0.png)
		- 3、原型设计模式
		  collapsed:: true
			- into函数中 用到RequestObtions的clone
		- 4、建造者模式
	- ## 3、项目中大量的使用了Glide，偶尔会出现内存溢出问题，请说说大概是什么原因？
		- 1、尽量在with的时候，传入有生命周期的作用域(非Application作用域)
		- 2、尽量避免使用了Application作用域，因为Application作用域不会对页面绑定生命周期机制，导致回收不及时，释放操作等....
	- ## 4、使用Glide为什么要加入网络权限？ <uses-permission android:name="android.permission.INTERNET" />
		- 答：等待队列/运行队列 执行Request ---> 活动缓存 --->内存缓存 ---> jobs.get检测执行的任务有没有执行完成 ---> HttpUrlFetcher.HttpURLConnection
	- ## 5、使用Glide时，with函数在子线程中，会有什么问题？
		- 答：子线程，不会去添加 生命周期机制， 主线程才会添加 空白的Fragment 去监听 Activity Fragment 的变化。
	- ## 6、使用Glide时，with函数传入Application后，Glide内部会怎么处理？
		- 1、在MainActivity中，MainActivity销毁了，并会让Glide生命周期机制处理回收，
		- 2、传入Application，会产生Application作用域，只有在整个APP应用都没有的时候，跟随着销毁（上节课 ApplicationLIfecycle add onStart  onDestroy 什么事情都没有做）。
- # 二、源码题
	- ## 3、怎么确保一个Activity只有一个空白Fragment监听生命周期
	  collapsed:: true
		- ## [[怎么确保一个Activity只有一个Fragment]]
	- ## 4、[[空白Fragment感知生命周期流程]]
	- ## 10、[[Glide缓存机制]]
- # 三、自定义一个图片框架都可以包含哪些内容
	- 1、比如Glide基本的加载能力，监听宿主生命周期、缓存
	- 2、提供监控，比如监控成功失败，Bitmap大小等