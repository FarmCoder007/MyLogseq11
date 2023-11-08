# Handler怎么避免内存泄漏#Card
	- # 泄漏写法:匿名内部类
		- ```java
		      Handler mHandler = new Handler(){
		          @Override
		          public void handleMessage(@NonNull Message msg) {
		              super.handleMessage(msg);
		          }
		      };
		  
		   mHandler.sendMessage(Message.obtain());
		  
		  ```
	- # 解决方案1：静态内部类+弱引用
		- ```java
		  public class MyActivity extends Activity {
		      private static class MyHandler extends Handler {
		          private WeakReference<MyActivity> mActivityRef;
		  
		          public MyHandler(MyActivity activity) {
		              mActivityRef = new WeakReference<>(activity);
		          }
		  
		        // 处理消息的逻辑
		  
		          @Override
		          public void handleMessage(@NonNull Message msg) {
		              super.handleMessage(msg);
		          }
		      }
		  
		      private MyHandler mHandler = new MyHandler(this);
		      mHandler.sendMessage(Message.obtain());
		      // ...
		  }
		  
		  ```
	- # 解决方案2：Activity onDestroy时，移除所有消息
		-
		- ```java
		  public class MyActivity extends Activity {
		    private Handler mHandler = new Handler();
		  - @Override
		    protected void onDestroy() {
		        super.onDestroy();
		        mHandler.removeCallbacksAndMessages(null);
		    }
		  - // ...
		  }
		  ```
-