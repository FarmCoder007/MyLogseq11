- # Android系统服务拦截概念
  collapsed:: true
	- Shadow模块代理了android系统的service，可以实现ShadowServiceInterceptor接口拦截系统服务，实现自定义逻辑，整体结构如下所示：
		- ![image.png](../assets/image_1684425689542_0.png)
- # 如何使用Shadow库拦截Android系统服务？
	- 下面我们将以Android系统的TelephonyManager服务的getDeviceId()方法为例，一步步实现服务拦截。
- # 0x00 添加Shadow依赖项
  collapsed:: true
	- 添加Shadow和FreeReflection依赖到项目中，其中Shadow提供拦截系统服务功能，FreeReflection则用于解决访问Android系统的hidden-apis。
	- ```
	  allprojects {
	      repositories {
	          maven { url "https://jitpack.io" }
	      }
	  }
	  
	  dependencies {
	      implementation 'com.github.tiann:FreeReflection:3.1.0'
	      implementation 'com.coofee:shadow:0.0.9'
	  }
	  ```
- # 0x01 初始化Shadow库
	- 在Application的attachBaseContext()方法中，初始化Shadow库，模板代码如下：
	  collapsed:: true
		- ```
		  public class App extends Application {
		  
		      @Override
		      protected void attachBaseContext(Context base) {
		          super.attachBaseContext(base);
		          initShadowManager(base);
		      }
		  
		      private void initShadowManager(Context base) {
		          if (Reflection.unseal(base) != 0) {
		              Log.e(ShadowServiceManager.TAG, "fail Reflection.unseal().");
		              return;
		          }
		          Log.e(ShadowServiceManager.TAG, "success Reflection.unseal().");
		  
		          ShadowConfig.Builder shadowConfigBuilder = new ShadowConfig.Builder(base, this)
		                     .debug(BuildConfig.DEBUG)
		                     .devTools(ShadowDevTools.DEFAULT_DEV_TOOLS)
		                    .logMode(ShadowLog.DEBUG)
		                    .interceptAll(true)
		  //                .addPackageOrClassNamePrefix("com.coofee.shadowapp")
		                    .add(new ServiceInterceptor())
		          ;
		  
		          ShadowServiceManager.init(shadowConfigBuilder.build());
		      }
		  }
		  
		  ```
	- ## 1. ShadowConfig说明
	  collapsed:: true
		- ShadowConfig的配置仅对通过add方法添加的Interceptor生效
		- debug方法用于设置deubg模式; debug模式下devTools生效。
		  devTools方法可以设置ShadowDevTools, 仅在debug模式下生效，当有服务方法被调用时，就会被触发，可以用来监控服务方法调用。
		  logMode方法指定日志开关。
		  logImpl方法指定日志实现类，可以更换为自己的实现类，默认会打印日志到控制台，日志tag为ShadowServiceManager。
		  add方法添加针对Service的拦截器，在拦截器的实现中会指定Service的名字。
		  interceptAll为true时，所有已添加的拦截器生效; 默认值为false。
		  addPackageOrClassNamePrefix指定包名前缀列表，已添加的拦截器只拦截包含包名前缀的Service调用(通过运行期间分析Service的调用栈，当Service的调用栈包含任一指定包名前缀时，则使用拦截器进行拦截)；默认为空列表。
		  注意：
		  当interceptAll为false，同时addPackageOrClassNamePrefix未添加包名前缀列表时，拦截器不生效。
		  当intercepAll为true时，会忽略addPackageOrClassNamePrefix添加的包名前缀列表，拦截器生效。
	- ## 2. ShadowServiceManager说明
	  collapsed:: true
		- ShadowServiceManager只拦截通过ShadowConfig添加的拦截器中指定的系统服务，通过init方法传入ShadowConfig进行初始化。
	- ## 3. ShadowServiceInterceptor说明
	  collapsed:: true
		- 在Shadow中，所有的拦截器均需要实现ShadowServiceInterceptor接口，
			- ```
			  public interface ShadowServiceInterceptor {
			  
			      // 拦截系统服务方法
			      Object invoke(String serviceName, Object service, Method method, Object[] args) throws Throwable;
			  
			      // 指定待拦截的系统服务名称
			      String provideInterceptServiceName();
			  
			      // 指定拦截的系统服务方法名称
			      Set<String> provideInterceptMethodNames();
			  
			      // true=拦截指定系统服务的全部方法; 默认为false，此时会使用`provideInterceptMethodNames()`返回值确定拦截的方法。
			      default boolean interceptAllMethod() {
			          return false;
			      }
			  }
			  ```