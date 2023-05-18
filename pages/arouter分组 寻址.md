- ARouter是以路由的方式实现组件间（组件化项目）通讯的的框架。
- 路由的本质，是映射和寻址，收集所有的注册类，生成字符串和注册类的映射关系，这样就可以通过字符串找到对应的类.
- 解决的问题，没有依赖关系的两个模块，不能直接交互，路由提供了仓库，可以通过字符串找到注入仓库的类，解决类模块间交互的问题（组件间通讯）
- 如何分组和构建路由表的呢？
- ## 一、从使用说起
  collapsed:: true
	- 1、我们按照文档使用ARouter 的时候注意到， 使用@Route注解的模块，需要在modeule 的build.gradle里添加：
	  collapsed:: true
		- ```
		  defaultConfig {
		     javaCompileOptions {
		         annotationProcessorOptions {
		             arguments = [AROUTER_MODULE_NAME: project.getName()]
		         }
		     }
		  }
		  ```
		- 没有这段代码会在build项目时爆错：
		  These no module name, at ‘build.gradle’, like : …
	- 2、必须在每个使用@Route 注解的模块里都引入ARouter的注解处理器，否则这个模块里注解不会被处理
	  collapsed:: true
		- ```
		  annotationProcessor 'com.alibaba:arouter-compiler:1.5.2'
		  ```
	- 3、@Route 注解的path 至少需要有两级
	  collapsed:: true
		- ```
		  @Route(path = "/test/activity")
		  public class YourActivity extend Activity {
		      ...
		  }
		  ```
		- 否则toast提示：“There’s no route matched! Path = [/xxx/xxx] Group = [xxxx]”
		- 编译生成的类：
		  collapsed:: true
			- ![image.png](../assets/image_1684399289080_0.png)
		- root
		  collapsed:: true
			- ![image.png](../assets/image_1684399297749_0.png)
		- group
		  collapsed:: true
			- ![image.png](../assets/image_1684399307028_0.png)
		- ARouter$Root$$xxx(modulename) 把所有的组(ARouter$Group$xxx) put到Map集合里（routers）
		  
		  * ARouter$Group$$xxx(groupname) 把一个分组下的所有路径(RouteMeta)存入map
		  
		  * ARouter$Providers$$xxx(modulename) 把注册的接口存入map
	-
	-
- ## 二.ARouter 注解处理器:RouteProcessor
	- 有注解就有注解处理器，ARouter也是基于APT+ Javapoat，构建路由表的逻辑就在RouteProcessor，也是在RouteProcessor里生成了上面的那些类
	- APT 和 javapoat 有同学分享过，这也是APT 和 javapoat应用很好的一个例子
	- 1、BaseProcessor
		- RouteProcessor 继承了 BaseProcessor
		  collapsed:: true
			- ```
			  public abstract class BaseProcessor extends AbstractProcessor {
			       ...
			       // 模块名
			       String moduleName = null;
			       //是否需要生成router 文档
			       boolean generateDoc;
			  
			       @Override
			       public synchronized void init(ProcessingEnvironment processingEnv) {
			           super.init(processingEnv);
			           //初始化工具类
			           mFiler = processingEnv.getFiler();
			           types = processingEnv.getTypeUtils();
			           elementUtils = processingEnv.getElementUtils();
			           typeUtils = new TypeUtils(types, elementUtils);
			           logger = new Logger(processingEnv.getMessager());
			  
			           // Attempt to get user configuration [moduleName]
			           Map<String, String> options = processingEnv.getOptions();
			           if (MapUtils.isNotEmpty(options)) {
			  
			               //从options里获取 moduleName
			               moduleName = options.get(KEY_MODULE_NAME);
			               generateDoc = VALUE_ENABLE.equals(options.get(KEY_GENERATE_DOC_NAME));
			           }
			  
			           if (StringUtils.isNotEmpty(moduleName)) {
			               moduleName = moduleName.replaceAll("[^0-9a-zA-Z_]+", "");
			  
			           } else {
			               //moduleName为空抛出异常，停止编译
			               logger.error(NO_MODULE_NAME_TIPS);
			              。。。
			           }
			       }
			  
			       ...
			  
			       @Override
			       public Set<String> getSupportedOptions() {
			           return new HashSet<String>() {{
			               this.add(KEY_MODULE_NAME);
			               this.add(KEY_GENERATE_DOC_NAME);
			           }};
			       }
			  }
			  ```
		- 主要初始化工具类，从gradle 配置里获取 moduleName
		-
	-
-