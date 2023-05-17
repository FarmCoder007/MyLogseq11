- 一、简介
	- HTTP是现代应用程序用来交换数据和媒体的网络方式。高效地执行HTTP能让资源加载更快并节省带宽。
	  OkHttp是一个高效的HTTP客户端，具有一下默认特性：
	- 支持HTTP/2，允许请求共用同一个socket连接
	  连接池可以减少请求延迟
	  透明的GZIP压缩减小了响应数据的大小
	  缓存响应内容，避免完全重复的请求
- 二、OkHttpClient
  collapsed:: true
	- OkHttp对外提供了 OkHttpClient 类来配置网络请求的参数以及执行网络请求
	- 在使用中，[[#red]]==OkHttpClient 可以使用单例也可以不使用单例==，对此官方并未有特别的指导说明
	- 这两种方式的主要区别在于 OkHttpClient 对象的创建，对于单例模式，每个请求都使用同一个 OkHttpClient 对象，毫无疑问在构建 OkHttpClient 对象的速度上单例模式更有优势，当然这种速度上的优势并不明显。
	- 除此之外，主要在于线程的开销上面，创建多个 OkHttpClient 对象是否会加大线程开销？
	- 带着这个疑问我们来看下 OkHttp 中的线程
- 三、OkHttp的线程池
	- 网络请求的示例代码如下：
	  collapsed:: true
		- ```kotlin
		  OkHttpClient client = new OkHttpClient();
		  Request request = new Request.Builder()
		      .url(url)
		      .build();
		  
		  // 同步请求
		  Response response = client.newCall(request).execute()
		  
		  // 异步请求
		  client.newCall(request).enqueue(new Callback() {
		      @Override
		      public void onResponse(@NotNull Call call, @NotNull Response response) throws IOException {
		      }
		  
		      @Override
		      public void onFailure(@NotNull Call call, @NotNull IOException e) {
		      }
		  });
		  ```
	- OkHttpClient 通过调用 newCall(Request) 方法生成执行网络请求的 Call，Call 类是一个接口，实际执行网络请求的 RealCall 类
	- Call 提供了同步和异步两种请求方式的API，同步请求会在当前线程直接执行网络请求，不会开新线程，当然也不会使用线程池，我们可以忽略掉这种请求方式，只需要关注异步请求即可
	- ### OkHttp的异步请求的线程池
		- OkHttp的异步请求流程如下图：
		  collapsed:: true
			- ![image.png](../assets/image_1684314059277_0.png)
		- 从请求流程中可以看出，OkHttp提供了 Dispatcher 类来对请求进行调度，Dispatcher 默认提供了一个线程池，请求会在该线程池中执行，同时 OkHttpClient 提供了一个指定 Dispatcher 的API，调用方可以指定线程池
		- Dispatcher 类进行请求调度的关键代码如下：
		  collapsed:: true
			- ```kotlin
			  private boolean promoteAndExecute() {
			    List<AsyncCall> executableCalls = new ArrayList<>();
			    boolean isRunning;
			    synchronized (this) {
			      // 遍历请求等待队列
			      for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
			        AsyncCall asyncCall = i.next();
			        // 正在执行的请求数量已达最大限制，不再执行新请求
			        if (runningAsyncCalls.size() >= maxRequests) break; // Max capacity.
			        // 同Host下正在执行的请求数量已达最大限制，不再执行新请求
			        // 实际最大可执行的数量为maxRequestsPerHost + 1
			        if (runningCallsForHost(asyncCall) >= maxRequestsPerHost) continue; // Host max capacity.
			  
			        i.remove();
			        // 可以执行的请求放入executableCalls，并添加到正在执行的请求队列
			        executableCalls.add(asyncCall);
			        runningAsyncCalls.add(asyncCall);
			      }
			      isRunning = runningCallsCount() > 0;
			    }
			    // 执行请求
			    for (int i = 0, size = executableCalls.size(); i < size; i++) {
			      AsyncCall asyncCall = executableCalls.get(i);
			      asyncCall.executeOn(executorService());
			    }
			    return isRunning;
			  }
			  ```