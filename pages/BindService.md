# 一、api
	- 参数一、Intent
		- 它和startService()里面那个intent是一样的，用来指定启动哪一个service以及传递一些数据过去。
	- 参数二、ServiceConnection
	  collapsed:: true
		- ```java
		  ServiceDemo mService;
		  //BinderDemo是在ServiceDemo里面的一个继承了Binder的内部类，这是一种得到IBinder接口的方式
		  //下文会有详述
		  ServiceDemo.BinderDemo mBinder;
		  
		  private ServiceConnection mConnection = new ServiceConnection() {
		  
		      //系统会调用该方法以传递服务的　onBind() 方法返回的 IBinder。
		      @Override
		      public void onServiceConnected(ComponentName name, IBinder service) {
		          //当系统调用 onServiceConnected() 回调方法时，我们可以使用接口定义的方法开始调用服务。
		          mBinder = (ServiceDemo.BinderDemo) service;
		          //getService()是BinderDemo中的一个方法
		          mService = mBinder.getService();
		          //在此处可以利用得到的ServiceDemo对象调用该类中的构造方法
		          Log.d(this.getClass().getSimpleName(), "onServiceConnected");
		      }
		  
		      //Android系统会在与服务的连接意外中断时（例如当服务崩溃或被终止时）调用该方法。当客户端取消绑定时，系统“绝对不会”调用该方法。
		      @Override
		      public void onServiceDisconnected(ComponentName name) {
		          Log.d(this.getClass().getSimpleName(), "onServiceDisconnected");
		      }
		  };
		  ```
	- 参数三、BIND_AUTO_CREATE，以便创建尚未激活的服务
		- BIND_DEBUG_UNBIND 和 BIND_NOT_FOREGROUND，或 0（表示无）。
	- ```
	  public boolean bindService(Intent service, ServiceConnection conn, int flags) {
	      return mBase.bindService(service, conn, flags);
	  }
	  ```
- # 二、使用
	- 一是在客户端完成bindService的调用以及相关配置
		- 客户端代码
			- ```java
			  public class BindingActivity extends Activity {
			      LocalService mService;
			      boolean mBound = false;
			  
			      @Override
			      protected void onCreate(Bundle savedInstanceState) {
			          super.onCreate(savedInstanceState);
			          setContentView(R.layout.main);
			      }
			  
			      @Override
			      protected void onStart() {
			          super.onStart();
			          // Bind to LocalService
			          Intent intent = new Intent(this, LocalService.class);
			          bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
			      }
			  
			      @Override
			      protected void onStop() {
			          super.onStop();
			          // Unbind from the service
			          if (mBound) {
			              unbindService(mConnection);
			              mBound = false;
			          }
			      }
			  
			      /** Called when a button is clicked (the button in the layout file attaches to
			        * this method with the android:onClick attribute) */
			      public void onButtonClick(View v) {
			          if (mBound) {
			              // Call a method from the LocalService.
			              // However, if this call were something that might hang, then this request should
			              // occur in a separate thread to avoid slowing down the activity performance.
			              int num = mService.getRandomNumber();
			              Toast.makeText(this, "number: " + num, Toast.LENGTH_SHORT).show();
			          }
			      }
			  
			      /** Defines callbacks for service binding, passed to bindService() */
			      private ServiceConnection mConnection = new ServiceConnection() {
			  
			          @Override
			          public void onServiceConnected(ComponentName className,
			                  IBinder service) {
			              // We've bound to LocalService, cast the IBinder and get LocalService instance
			              LocalBinder binder = (LocalBinder) service;
			              mService = binder.getService();
			              mBound = true;
			          }
			  
			          @Override
			          public void onServiceDisconnected(ComponentName arg0) {
			              mBound = false;
			          }
			      };
			  }
			  ```
	- 二是在服务端里面实现onBind()方法的重写，返回一个用做信息交互的IBinder接口
		- 获取IBinder三种方式：继承Binder类，使用Messenger类，使用AIDL
		- 服务端代码
			- ```java
			  public class LocalService extends Service {
			      // Binder given to clients
			      private final IBinder mBinder = new LocalBinder();
			      // Random number generator
			      private final Random mGenerator = new Random();
			  
			      /**
			        继承的Binder类
			       * Class used for the client Binder.  Because we know this service always
			       * runs in the same process as its clients, we don't need to deal with IPC.
			       */
			      public class LocalBinder extends Binder {
			          LocalService getService() {
			              // Return this instance of LocalService so clients can call public methods
			              return LocalService.this;
			          }
			      }
			  
			      @Override
			      public IBinder onBind(Intent intent) {
			          return mBinder;
			      }
			  
			      /** method for clients */
			      public int getRandomNumber() {
			        return mGenerator.nextInt(100);
			      }
			  }
			  ```
-