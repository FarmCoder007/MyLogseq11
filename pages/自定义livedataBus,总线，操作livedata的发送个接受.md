## livedataBus
	- ```java
	  public class LiveDataBus {
	      //存放订阅者
	      private Map<String, MutableLiveData<Object>> bus;
	      private static LiveDataBus liveDataBus = new LiveDataBus();
	  
	      private LiveDataBus() {
	          bus = new HashMap();
	      }
	      public static LiveDataBus getInstance() {
	          return liveDataBus;
	      }
	      //注册订阅者
	      public synchronized <T> MutableLiveData<T> with(String key, Class<T> type) {
	          if(!bus.containsKey(key)){
	              bus.put(key,new MutableLiveData<Object>());
	          }
	          return (MutableLiveData<T>)bus.get(key);
	      }
	  }
	  ```
- ## 使用总线发送
	- ```java
	             new Thread(){
	                  @Override
	                  public void run() {
	                      for (int i = 0; i < 10; i++) {
	                          //发送消息
	                          LiveDataBus.getInstance().with("data", String.class).postValue("jett");
	                          try {
	                              Thread.sleep(5000);
	                          } catch (InterruptedException e) {
	                              e.printStackTrace();
	                          }
	  
	  
	                      }
	                  }
	              }.start();
	  
	  ```
- ## 使用总线接受
	- ```java
	         LiveDataBus.getInstance().with("data", String.class)
	                      .observe(this, new Observer<String>() {
	                          @Override
	                          public void onChanged(String s) {
	                              if(s!=null)
	                              Toast.makeText(TestLiveDataBusActivity.this, s, Toast.LENGTH_SHORT).show();
	                          }
	                      });
	  ```