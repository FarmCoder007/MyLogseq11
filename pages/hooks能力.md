- ## hook简介
	- “hook”一般指的是钩子函数（hook function）或钩子程序（hook program），是指在程序执行期间，通过更改或扩展现有代码的行为方式，以实现特定功能的技术手段。在软件开发中，钩子函数通常被用于实现各种插件系统、扩展系统、调试工具等功能。
	- 在Java语言中，常常使用代理模式来实现对现有代码的“hook”（钩子）功能。代理模式是一种结构型设计模式，它允许你提供一个代理类来替代原始的对象。代理类可以控制对原始对象的访问，并在原始对象的基础上提供一些额外的功能，比如拦截方法调用、添加日志、添加缓存等。
- ## 一、需求背景，hook功能api，批量处理
	- ![image.png](../assets/image_1683360102737_0.png)
- ## 二 Hook责任划分
	- ## 统一日志库SDK
		- 1、提供[[#red]]==Hook类-定义Hook范围==:
		  collapsed:: true
			- Hooks：
			  collapsed:: true
				- ```kotlin
				  package com.wuba.unitylog.hooks
				  
				  /**
				   *  作用：Hooks 类  对日志api打印的数据 做钩子处理，数据拼接好后拿过来 经过闭包hook函数 再处理一遍发出去
				   *  1、定义hook能力范围api，比如对所有日志生效的hook api 1,还是对指定tag生效的hook能力。只定义生效范围。
				   *    API ：
				   *       1、log（logHook: (log: String) -> String?） 是hook所有log打印数据的
				   *       2、log (tag: String , logHook: (log: String) -> String?) 是hook指定tag打印数据的
				   *  2、存储自定义日志组件传过来的闭包（对日志数据的具体处理逻辑，如拼接哪些参数）
				   *        具体hook逻辑是各个接入方 在日志组件里自定义的闭包传过来的：
				   *
				   */
				  class Hooks {
				      /**
				       *  存储组件传过来的hook能力，按道理 通常就一个通用的因为加参数可以拼接
				       */
				      val logHooks: MutableList<(log: String) -> String?> = mutableListOf()
				  
				      /**
				       *  按 tag标签 存储 hook能力（闭包 传入参数 包装处理后 再返回）
				       */
				      val tagLogHooks: HashMap<String , (log: String) -> String?> = HashMap()
				  
				      /**
				       *  hook通用log能力，调用此方法一次，代表添加一项hook能力到列表中。
				  
				       *  @param logHook闭包：具体的hook逻辑
				       *  (看组件实现，比如接受拼接好的log参数，拼接一些通用参数再返回)
				       */
				      fun log(logHook: (log: String) -> String?) {
				          if (!logHooks.contains(logHook)) {
				              logHooks.add(logHook)
				          }
				      }
				  
				      /**
				       *  处理指定tag对应log  hook的能力
				       *  @param tag  筛选对应tag,传入的hook闭包能力。可以不同tag，对应不同的hook能力比如 不同tag
				       */
				      fun log(tag: String , logHook: (log: String) -> String?) {
				          if (!tagLogHooks.containsKey(tag)) {
				              tagLogHooks[tag] = logHook
				          }
				      }
				  }
				  ```
			- 作用：
				- 1、定义hook能力范围api，比如对所有日志生效的hook api 1,还是对指定tag生效的hook能力。只定义生效范围。
				- 2、存储自定义日志组件传过来的闭包（对日志数据的具体处理逻辑，如拼接哪些参数）
				       具体hook逻辑是各个接入方 在日志组件里自定义的闭包传过来的：
		- 2、[[#red]]==UnityLogSDK：SDK初始化类-注册Hook对象==
		  collapsed:: true
			- 代码示例：
			  collapsed:: true
				- ```kotlin
				  package com.wuba.unitylog
				  
				  import android.content.Context
				  import com.wuba.unitylog.hooks.Hooks
				  import java.lang.ref.WeakReference
				  
				  object UnityLogSDK {
				  
				      /**
				       * 是否已初始化
				       */
				      @Volatile
				      private var isInit: Boolean = false
				  
				      /**
				       * Application context
				       */
				      private lateinit var applicationContext: Context
				  
				      /**
				       * 是否为发布包
				       */
				      var isPublishPackage: Boolean = false
				          private set
				  
				      /**
				       * flipper 输出接口
				       */
				      internal var unityFlipper: IUnityFlipper? = null
				  
				      /**
				       * wlog 输出接口
				       */
				      internal var unityWLog: IUnityWLog? = null
				  
				      /**
				       * 每个配置类的 key对应一个hook对象
				       */
				      private val hookObjects = HashMap<String, WeakReference<Hooks>>()
				  
				      @JvmStatic
				      fun init(applicationContext: Context, config: Config): UnityLogSDK {
				          if (isInit) {
				              return this
				          }
				  
				          this.applicationContext = applicationContext
				          isPublishPackage = config.isPublishPackage
				          unityFlipper = config.unityFlipper
				          unityWLog = config.unityWLog
				  
				          isInit = true
				  
				          return this
				      }
				  
				      /**
				       *  注册Hook对象 到Map
				       
				       *  根据主键 获取map中 组件的hook对象，
				       *  有则返回
				       *  无则添加后返回
				       */
				      fun getHook(logKey: String): Hooks? {
				          return try {
				              hookObjects[logKey]?.get() ?: Hooks().also {
				                  hookObjects[logKey] = WeakReference(it)
				              }
				          } catch (e: Exception) {
				              null
				          }
				      }
				  
				      // 在日志打印类中 获取注册的 hook对象，调用里边添加的hook逻辑
				      fun getHooks() = hookObjects
				  }
				  ```
			- 注册时机：这个注册Hook对象是在接入方的，打印日志类LOGGER自动完成的
			- 该类作用：Map存储注册的Hook对象，按主键一个模块一个Hook对象
		- 3、[[#red]]==SDK的日志打印类-使用注册的Hook对象==调用hook能力处理：
		  collapsed:: true
			- 具体打印日志逻辑中，拿到注册到SDK的所有 hook对象 [因为不同SDK的主键不一样，hook对象也不一样，个性化处理]
			- 部分代码：
				- ```kotlin
				  
				  fun makeFullMessage(recycleLog: RecycleLog, builder: StringBuilder, params: String?): String {
				              // 省略了处理拼接字符串     
				       
				              // logBody：拼接后的日志参数
				              var logBody = builder.toString()
				              // 遍历注册的hook对象【可能每个使用方SDK 一个 - 多个】
				              UnityLogSDK.getHooks().values.forEach {
				                  // 调用 使用方添加的hook闭包 处理字符串 并返回
				                  it.get()?.logHooks?.forEach { hook ->
				                      logBody = hook.invoke(logBody).toString()
				                  }
				                  // 按 tag 处理 hook
				                  it.get()?.tagLogHooks?.forEach { (key, hook) ->
				                      // 根据 tag过滤
				                      if (recycleLog.tag == key) {
				                          logBody = hook.invoke(logBody).toString()
				                      }
				                  }
				              }
				              return logBody
				  }
				  ```
	- ## 接入方
		- 1、新增日志组件 继承 日志库SDK的日志组件类
		- 2、实现注册hook的方法，内部自定义处理hook逻辑
			- 1、 根据主Key注册 hook对象 到日志SDK中
			   2、定义hook能力 闭包【其实就是处理函数】，会被存储到 刚刚创建的hook对象中
			   3、等真正打印日志的时候 再去遍历注册的hook对象。循环调用 hook能力
			- 代码：
			  collapsed:: true
				- ```kotlin
				  @UnityLogConfig(className = "DemoUnityLogger", primaryKey = "UnityLogSDK")
				  open class DemoUnityLoggerComponent : UnityLogComponent() {
				      // override
				      override fun hooks(unityLog: UnityLogSDK) {
				          // 1、 根据主Key注册 hook对象 到日志SDK中
				          // 2、定义hook能力 闭包【其实就是处理函数】，会被存储到 刚刚创建的hook对象中
				          // 3、等真正打印日志的时候 再去遍历注册的hook对象。循环调用 hook能力
				          unityLog.getHook(getPrimaryTag())?.log { log: String ->
				              "$log ****** ppu=${this.getPPU()}"
				          }
				          unityLog.getHook(getPrimaryTag())?.log("tag") { log: String ->
				              "$log ****** ppu=${this.getPPU()}"
				          }
				      }
				  
				      fun getPPU(): String {
				          return "123"
				      }
				  }
				  ```
			- 具体hook逻辑 就体现这个传入的闭包中
				- ```kotlin
				  hook的api：
				     fun log(logHook: (log: String) -> String?) {
				          if (!logHooks.contains(logHook)) {
				              logHooks.add(logHook)
				          }
				      }
				     
				     // 调用的地方传入的闭包
				  unityLog.getHook(getPrimaryTag())?.log { log: String ->
				              "$log ****** ppu=${this.getPPU()}"
				          }
				  
				   // 
				   { log: String ->  "$log ****** ppu=${this.getPPU()}" }
				  
				  ```
- ## 三、实现
	- ### 框架梳理见[[日志库接入流程]]
	-