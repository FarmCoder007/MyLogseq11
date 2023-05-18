- # 简介
	- 目前大多数项目都采用的Retrofit作为自己的网络框架，而自己对于该框架的掌握程度还停留在CV阶段。是时候搞一波源码分析了！
	- Retrofit最让大家称道的是他可以让请求兼容rxJava，也可以轻松的将网络请求结果转换成自己想要的实体。那它究竟是怎么实现的呢？下来我们就扒拉下他的源码看看他是怎么实现这么牛逼的功能的。
- # 使用示例
	- 老样子，我们从一个简单的使用例子着手，逐步深入往里看。
	  collapsed:: true
		- ```
		  OkHttpClient.Builder httpClient = new OkHttpClient.Builder();
		          Retrofit retrofit = new Retrofit.Builder()
		                  .addConverterFactory(GsonConverterFactory.create(new GsonBuilder().create()))
		                  .addCallAdapterFactory(RxJava2CallAdapterFactory.createAsync())
		                  .baseUrl("https://api.douban.com/v2/").client(httpClient.build())
		                  .build();
		          BookService service = retrofit.create(BookService.class);
		  ```
	- 首先通过builder传入ConverterFactory，传入CallAdapterFactory。
	- 然后将OkHttpClient对象也塞了进去。
	- 最后调用create方法传入请求服务的接口，就可以构造出一个请求服务的实例对象。
-
- 参考：
	- https://hexilee.me/2018/09/27/animal-sniffer/