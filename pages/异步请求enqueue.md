- ```java
      @Override
      public void enqueue(Callback responseCallback) {
          synchronized (this) {
              if (executed) throw new IllegalStateException("Already Executed");
              executed = true;
          }
          captureCallStackTrace();
          eventListener.callStart(this);
  		//调用分发器
          client.dispatcher().enqueue(new AsyncCall(responseCallback));
      }
  ```
-