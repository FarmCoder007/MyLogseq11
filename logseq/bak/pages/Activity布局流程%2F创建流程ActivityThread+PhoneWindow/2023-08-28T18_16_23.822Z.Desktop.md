- [[#red]]==**Activity 整体布局结构**==
  collapsed:: true
	- ![image.png](../assets/image_1691071466517_0.png)
- ## 1、ActivityThread调用performLaunchActivity启动Activity时
  collapsed:: true
	- 1、[[#red]]==**创建上下文**==
	  collapsed:: true
		- ```java
		   ContextImpl appContext = createBaseContextForActivity(r);
		  ```
	- 2、[[#red]]==**通过mInstrumentation**==.newActivity[[#red]]==**创建 Activity**==
	- 3、[[#red]]==**创建WINDOW**==：window = r.mPendingRemoveWindow;，此时window还为空
	- 4、activity.[[#red]]==**attach**==(appContext, [[#red]]==**建立activity与window的关联**==
	  collapsed:: true
		- 实际上再Activity类中创建出PhoneWindow
		- Activity.java中   mWindow = new PhoneWindow(this, window, activityConfigCallback);
	- 5、mInstrumentation调用[[#red]]==**Activity的onCreate生命周期方法**==
	  collapsed:: true
		- mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
- ## 2、Activity-onCreate-走setContentView
  collapsed:: true
	- ```java
	      protected void onCreate(Bundle savedInstanceState) {
	          super.onCreate(savedInstanceState);
	          setContentView(R.layout.activity_rx);
	  
	  ```
- ## 3、PhoneWindow（由上，调用到PhoneWindow的setContentView）
  collapsed:: true
	- ## 1、installDecor 初始化[[#red]]==**DecorView（FrameLayout）**==
		- 1、往DecorView添加默认布局LinerLayout,
		- 2、从上边默认布局中，根据[[#red]]==**id = content 初始化mContentParent变量**==
	- ## 2、mLayoutInflater.inflate(layoutResID, mContentParent);
		- inflate我们activity 传入的布局id。父view为 initDecor中初始化的变量mContentParent也为一个Framelayout
- >[[#red]]==**下边就是解析xml布局文件，反射创建布局里的view[插件化换肤使用]**==
- ## 4、LayoutInflater.inflate
	- 1、[[createViewFromTag]]方法中 根据createView(...) 在这里使用[[#red]]==**反射**==调用2个参数的构造方法实例化view对象
	- > 所以自定义view一定重写双参构造
	- > app里所有的view 创建 都需要走LayoutInflater 的createView方法创建实例
	     1、createView方法
	     2、通过mFactory.onCreateView    【插件化换肤入口】
	- > [[#red]]==**由createViewFromTag方法内可知，根据xml创建的对象还没有布局参数，即LayoutParams**==
	- 2、root不为空：[[#red]]==**根据root生成布局参数**==params = root.generateLayoutParams(attrs);
	- 3、根据attachToRoot 设置 layoutParams  temp.setLayoutParams(params);
		- inflate xml 的根布局就是temp
		- ## [[LayoutInflater.inflate 第三个参数作用]]
- ## [[插件化换肤]]