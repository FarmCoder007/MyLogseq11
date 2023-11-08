# 1、分组group目的：
collapsed:: true
	- [[#red]]==**1、避免路由重复**==：路由使用二级目录，添加分组。每个模块都有可能叫mainActivity的，
	  collapsed:: true
		- 添加注解path:groupName/pathName
			- ```java
			  /order/MainActivity
			  ```
	- [[#red]]==**2、查找快捷**==，分层更加清晰,按组加载
	- [[#red]]==**3、延时加载，分摊首次加载时间**==，root表
		- 启动时加载root表
		- 第一次使用某个分组路由，加载该组group表
		- ### 参考
		  collapsed:: true
			- 启动应用，然后立马遍历所有com.alibaba.android.arouter.routes开头的文件，分析出是Root的文件，反射实例化，调用loadInto方法，将IRouteGroup.class载入。这几步里好多是耗时的方法，如果工程很庞大，像淘宝这样，如果一上来直接加载所有的IRouteGroup中的内容，恐怕要非常长的时间，会导致启动慢，或者ANR。
			- 这也是为什么ARouter需要IRouteRoot存在的原因，他可以延时加载所需要的分组，第一次使用的时候加载可以分摊加载时间。
- # 2、生成Root类名包含moduleName作用
  collapsed:: true
	- 按模块名moduleName生成root表`ARouter$Root$$xxx(modulename)` 。[[#green]]==**避免多模块下，生成root表类名冲突**==，没有moduleName每个模块生成一个ARouter$Root，类冲突
- # 3、不使用Arouter这种路由框架，使用全局Map注册可以不
	- 可以实现，但是用户太繁琐，每个类都要手动注册路由到路由表map中。Arouter已经将路由表处理好 了
	- ![image.png](../assets/image_1692589856285_0.png)
- # 4、不同模块怎么查找的Group表的？
	- APT生成的路由表都放入同一包名下  com.alibaba.android.arouter.routes，同时都实现了同一个接口
- # 5、[[Arouter简单使用流程]]
- # 6、[[Arouter的依赖注入]]
- # 7、[[Arouter工作流程]]
- # 8、[[分组与寻址原理]]
- # 9、[[自动注册Group表原理]]
- # 10、按类型跳转
	- 源码
	  collapsed:: true
		- ```java
		  private Object _navigation(final Postcard postcard, final int requestCode, final NavigationCallback callback) {
		          final Context currentContext = postcard.getContext();
		  
		          switch (postcard.getType()) {
		              case ACTIVITY:
		                  // Build intent
		                  final Intent intent = new Intent(currentContext, postcard.getDestination());
		                  intent.putExtras(postcard.getExtras());
		  
		                  // Set flags.
		                  int flags = postcard.getFlags();
		                  if (0 != flags) {
		                      intent.setFlags(flags);
		                  }
		  
		                  // Non activity, need FLAG_ACTIVITY_NEW_TASK
		                  if (!(currentContext instanceof Activity)) {
		                      intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
		                  }
		  
		                  // Set Actions
		                  String action = postcard.getAction();
		                  if (!TextUtils.isEmpty(action)) {
		                      intent.setAction(action);
		                  }
		  
		                  // Navigation in main looper.
		                  runInMainThread(new Runnable() {
		                      @Override
		                      public void run() {
		                          startActivity(requestCode, currentContext, intent, postcard, callback);
		                      }
		                  });
		  
		                  break;
		              case PROVIDER:
		                  return postcard.getProvider();
		              case BOARDCAST:
		              case CONTENT_PROVIDER:
		              case FRAGMENT:
		                  Class<?> fragmentMeta = postcard.getDestination();
		                  try {
		                      Object instance = fragmentMeta.getConstructor().newInstance();
		                      if (instance instanceof Fragment) {
		                          ((Fragment) instance).setArguments(postcard.getExtras());
		                      } else if (instance instanceof android.support.v4.app.Fragment) {
		                          ((android.support.v4.app.Fragment) instance).setArguments(postcard.getExtras());
		                      }
		  
		                      return instance;
		                  } catch (Exception ex) {
		                      logger.error(Consts.TAG, "Fetch fragment instance error, " + TextUtils.formatStackTrace(ex.getStackTrace()));
		                  }
		              case METHOD:
		              case SERVICE:
		              default:
		                  return null;
		          }
		  
		          return null;
		      }
		  ```
	- ## activity
		- 通过隐式意图，StartActivity实现的
	- ## Fragment
		- 反射创建对象，返回Fragment