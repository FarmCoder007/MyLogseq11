# 一、简介
collapsed:: true
	- OkHttp是当下Android使用最频繁的网络请求框架，由Square公司开源。Google在Android4.4以后开始将源码中的HttpURLConnection底层实现替换为OKHttp，同时现在流行的Retrofifit框架底层同样是使用OKHttp的。
- # 二、优点:
  collapsed:: true
	- 支持Http1、Http2、Quic以及WebSocket
	- 连接池复用底层TCP(Socket)，减少请求延时
	- 无缝的支持GZIP减少数据流量
	- 缓存响应数据减少重复的网络请求
	- 请求失败自动重试主机的其他ip，自动重定向
- # 三、使用流程 [[Okhttp请求简单使用]]
	- ## 图
		- ![image.png](../assets/image_1689851761928_0.png){:height 877, :width 555}
	- 在使用OkHttp发起一次请求时，对于使用者最少存在 OkHttpClient 、 Request 与 Call 三个角色。其中OkHttpClient 和 Request 的创建可以使用它为我们提供的 Builder （建造者模式）。而 Call 则是把 Request 交给 OkHttpClient 之后返回的一个已准备好执行的请求。
	- > 建造者模式：将一个复杂的构建与其表示相分离，使得同样的构建过程可以创建不同的表示。实例化OKHttpClient和Request的时候，因为有太多的属性需要设置，而且开发者的需求组合千变万化，使用建造者模式可以让用户不需要关心这个类的内部细节，配置好后，建造者会帮助我们按部就班的初始化表示对象
	- 同时OkHttp在设计时采用的[[#red]]==**门面模式**==，将整个系统的复杂性给隐藏起来，将子系统接口通过一个客户端OkHttpClient统一暴露出来。OkHttpClient 中全是一些配置，比如代理的配置、ssl证书的配置等。而 Call 本身是一个接口，我们获得的实现为: RealCall
		- ```java
		  static RealCall newRealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket)
		  {
		      // Safely publish the Call instance to the EventListener.
		      RealCall call = new RealCall(client, originalRequest, forWebSocket);
		      call.eventListener = client.eventListenerFactory().create(call);
		      return call;
		  }
		  ```
	- Call 的 execute 代表了同步请求，而 enqueue 则代表异步请求。
	- 两者唯一区别在于一个会直接发起网络请求，而另一个使用OkHttp内置的线程池来进行。这就涉及到OkHttp的任务分发器。
- # 四、OkhttpClient
  collapsed:: true
	- ## [[OkhttpClient可设置单例，也可不使用单例，带来的线程池问题？]]
	- 每个OkhttpClient 都会新建个 分发器里一个线程池
	- 每一个 OkHttpClient 对象都会生成一个链接池的管理类 也是一个线程池
- # 四、[[分发器Dispatcher-4.11.0]]
- # 五、[[OKhttp拦截器]]
- # 六、总结
  collapsed:: true
	- 整个OkHttp功能的实现就在这五个默认的拦截器中，所以先理解拦截器模式的工作机制是先决条件。
	-
	- 这五个拦截器分别为: 重试拦截器、桥接拦截器、缓存拦截器、连接拦截器、请求服务拦截器。每一个拦截器负责的工作不一样，就好像工厂流水线，最终经过这五道工序，就完成了最终的产品。
	-
	- 但是与流水线不同的是，OkHttp中的拦截器每次发起请求都会在交给下一个拦截器之前干一些事情，在获得了结果之后又干一些事情。整个过程在请求向是顺序的，而响应向则是逆序。
	-
	- 当用户发起一个请求后，会由任务分发起 Dispatcher 将请求包装并交给重试拦截器处理。
	- 1、重试拦截器在交出(交给下一个拦截器)之前，负责判断用户是否取消了请求；在获得了结果之后，会根据响应码判断是否需要重定向，如果满足条件那么就会重启执行所有拦截器。
	- 2、桥接拦截器在交出之前，负责将HTTP协议必备的请求头加入其中(如：Host)并添加一些默认的行为(如：GZIP压缩)；在获得了结果后，调用保存cookie接口并解析GZIP数据。
	- 3、缓存拦截器顾名思义，交出之前读取并判断是否使用缓存；获得结果后判断是否缓存。
	- 4、连接拦截器在交出之前，负责找到或者新建一个连接，并获得对应的socket流；在获得结果后不进行额外的处理。
	- 5、请求服务器拦截器进行真正的与服务器的通信，向服务器发送数据，解析读取的响应数据。
	-
	- 在经过了这一系列的流程后，就完成了一次HTTP请求！
- # 七、[[Okhttp代理]]
-
-
- # [[OKhttp面试题]]
- # 相关文章
	- [[OkHttp线程池相关源码分析]]
	- [[五大拦截器]]
	- [Okhttp源码解析一](https://blog.csdn.net/xuwb123xuwb/article/details/114639411)
	- [怎么处理https的证书的](https://www.6hu.cc/archives/72873.html)