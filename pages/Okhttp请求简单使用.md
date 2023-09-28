# 一、代码
	- ```java
	  private void testOkHttp() throws IOException {
	      //Step1
	      final OkHttpClient client = new OkHttpClient();
	      //Step2
	      final Request request = new Request.Builder()
	      						.url("https://www.google.com.hk").build();
	      //Step3
	      Call call = client.newCall(request);
	      //step4 发送网络请求，获取数据，进行后续处理
	      call.enqueue(new Callback() {
	          @Override
	          public void onFailure(Call call, IOException e) {
	            
	          }
	          @Override
	          public void onResponse(Call call, Response response) throws IOException {
	              Log.i(TAG,response.toString());
	              Log.i(TAG,response.body().string());
	          }
	      });
	  }
	  ```
- # 二、步骤
  collapsed:: true
	- **Step1****：创建****HttpClient****对象，**也就是构建一个网络类型的实例，一般会将所有的网络请求使用同一个单例对象。
	- **Step2****：构建****Request**，也就是构建一个具体的网络请求对象，具体的请求url，请求头，请求体等等。
	- **Step3****：构建请求****Call**，也就是将具体的网络请求与执行请求的实体进行绑定，形成一个具体的正式的可执行实体。
	- **Step4: ** 后面就进行网络请求了，然后处理网络请求的数据了。
-
- ## **OKhttp****的意义**：
	- OkHttp 是基于Http协议封装的一套请求客户端，虽然它也可以开线程，但根本上它更偏向真正的请求，跟HttpClient, HttpUrlConnection的职责是一样的。
- ## **Okhttp****的职责：**
	- OkHttp主要负责socket部分的优化，比如多路复用，buffffer缓存，数据压缩等等。
- # Okhttp给用户留下的问题：
	- 1）用户网络请求的接口配置繁琐，尤其是需要配置请求body，请求头，参数的时候；
	- 2）数据解析过程需要用户手动拿到responsbody进行解析，不能复用；
	- 3）无法适配自动进行线程的切换。
	- 那么这几个问题谁来解决？ 对，retrofifit！