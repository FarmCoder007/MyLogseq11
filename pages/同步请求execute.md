- ```java
  @Override
      public Response execute() throws IOException {
          synchronized (this) {
              if (executed) throw new IllegalStateException("Already Executed");
              executed = true;
          }
          captureCallStackTrace();
          eventListener.callStart(this);
          try {
  			//调用分发器
              client.dispatcher().executed(this);
  			//执行请求
              Response result = getResponseWithInterceptorChain();
              if (result == null) throw new IOException("Canceled");
              return result;
          } catch (IOException e) {
              eventListener.callFailed(this, e);
              throw e;
          } finally {
  			//请求完成
              client.dispatcher().finished(this);
          }
      }
  ```