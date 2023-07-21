- 一个接口的method 都有一个自己的ServiceMethod，这就意味着ServiceMethod是属于函数的
- 1、[[#red]]==**ServiceMethod其实是用来存储一次网络请求的基本信息的，比如Host、URL、请求方法等**==
- 2、[[#red]]==**同时ServiceMethod还会存储用来适配OkHttpCall对象的CallAdpater。**==
  collapsed:: true
	- ServiceMethod的build方法会解读传入的Method，
	- 首先ServiceMethod会在CallAdpaterFactory列表中寻找合适的CallAdapter来包装OkHttpCall对象，这一步主要是根据Method的返回参数来匹配的
	- 比如如果方法的返回参数是Call对象，那么ServiceMethod就会使用默认的CallAdpaterFactory来生成CallAdpater，
	- 而如果返回对象是RxJava的Obserable对象，则会使用RxJavaCallAdapterFactory提供的CallAdpater。
- 3、[[#red]]==**获得注解上配置的网络请求信息**==，比如请求方法、URL、Header等