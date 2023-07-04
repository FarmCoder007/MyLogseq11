- ```java
      private ServiceConnection connection = new ServiceConnection() {
          @Override
          public void onServiceConnected(ComponentName name, IBinder service) {
              Log.e("leo", "onServiceConnected: success");
              iPersonManager =  IPersonManager.Stub.asInterface(service);// proxy
          }
  
          @Override
          public void onServiceDisconnected(ComponentName name) {
              Log.e("leo", "onServiceDisconnected: success");
              iPersonManager = null;
          }
      };
  ```
- ## onServiceConnected 中的第二个参数IBinder，
	- 是客户端调用bindService时。服务端Service 回调onBind（）方法返回的IBinder对象
	-
	- ## 如果是AIDL中
		- 是服务端应用程序Service的 `onBind()` 方法中返回的实现了 AIDL 接口的 `Binder` 对象。
		- 这个 `Binder` 对象在被返回给客户端应用程序时，会被包装成一个 `IBinder` 对象，从而进行跨进程传输。
		- 客户端应用程序接收到这个 `IBinder` 对象后，可以将其转换回服务端定义的 AIDL 接口类型。（通过 
		  Stub.asInterface）这样客户端就可以通过调用 AIDL 接口中定义的方法与服务端进行通信和交互。