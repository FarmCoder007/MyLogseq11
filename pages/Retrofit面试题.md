# 概念题
	- ## 1、[[Retrofit动态代理的好处]]
		- 1、动态代理可以**==代理接口的所有请求==**，让**[[#red]]==所有的接口请求都走 invoke函数==**，
		- 2、从而**[[#green]]==将网络接口的参数配置归一化==**。
	- ## 2、[[Retrofit 优点]]
		- 1、==**使用注解对请求参数的适配@get等**==，Okhttp需自己添加，现在通过注解添加
		- 2、封装了okhttp.call进行请求，==**自动将响应切回主线程**==
		- 3、==**对数据结果的封装**==
	- ## 3、[[Retrofit封装性具体体现在哪里？]]
		- 1、Retrofit采用==**外观模式**==对Okhttp 表层使用进行了[[#red]]==**简化封装**==
		- 2、而==**ServiceMethod**==是将okhttp==**网络请求信息**==的，比如参数拼接，数据回调等[[#red]]==**封装成了call**==
	- ## 4、[[Retrofit设计模式的应用]]
		- #### 1、整个retrofit 采用的时[[#red]]==**外观模式**==。统一的调用创建网络请求接口实例和网络请求参数配置的方法
		- #### 2、Retrofit 创建时的callFactory，使用[[#red]]==**工厂方法设计模式**==
		- #### 3、Retrofit 实例使用[[#red]]==**建造者模式通过Builder类构建**==
		- #### 4、retrofit里面使用了[[#red]]==**动态代理**==来创建网络请求接口实例
		- #### 5、ExecuteCallBack 使用[[#red]]==**装饰者模式来封装callbackExecutor**==，用于完成线程的切换
		- #### 6、CallAdapter采用了[[#red]]==**适配器模式**== 为创建访问Call接口提供服务
		- #### 7、ExecutorCallbackCall 使用[[#red]]==**静态代理(委托) 代理了Call**==进行网络请求
		- #### 8、使用了**策略模式**对serviceMethod对象进行网络请求参数配置
- # 源码题
	- # 基本主流程原理
		- ```java
		  //step1 Retrofit初始化
		  Retrofit retrofit = new Retrofit.Builder()
		                      .baseUrl("https://www.wanandroid.com/")
		                      .addConverterFactory(GsonConverterFactory.create(new Gson()))
		                      .build();
		  //step2 retrofit.create 动态代理  反射创建代理对象
		  ISharedListService sharedListService = retrofit.create(ISharedListService.class);
		  //step3  代理对象进程方法调用 返回 Call
		  Call<SharedListBean> sharedListCall = sharedListService.getSharedList(2,1);
		  //step4 call 执行请求
		  sharedListCall.enqueue(object :Callback<String>{
		              override fun onResponse(call: Call<String>, response: Response<String>) {
		                  TODO("Not yet implemented")
		              }
		  
		              override fun onFailure(call: Call<String>, t: Throwable) {
		                  TODO("Not yet implemented")
		              }
		          })
		  ```
	- # step1、[[Retrofit初始化时做了什么事情]]
		- ### 1：初始化callFactory 为OkhttpClient。[[#red]]==**创建RealCall用**==
			- 后边创建Okhttp realCall用
		- ### 2：用handler封装的Executor[[#red]]==**做线程切换用**==
		- ### 3：设置默认CallAdapterFactory，提供默认的CallAdapter转换器
			- > 这里设置的默认的CallAdapter是 defaultCallAdapterFactory，这个类会被ServiceMethod调用，比如 代理类执行代码的时候会调用到动态代理的invoke 方法处，通过这个defaultCallAdapterFactory的Adapt 方法将 OkhttpCall 封装成 ExecutorCallBackCall进行返回
			- [[#red]]==**默认的Adapter把okhttpcall 适配成Retrofit call（叫ExecutorCallBackCall）**==
		- ### 4：初始化转换器converterFactories，[[#red]]==**对请求和返回数据进行转换处理**==
	- # step2、create创建代理对象[[Retrofit动态代理具体执行逻辑]]
		- ### 1、retrofit.create采用动态代理在**[[#green]]==内存中创建网络请求接口的代理类==**，[[#green]]==**反射创建代理类对象**==，
		- ### 2、代理类==构造函数==中会传入[[#red]]==**InvocationHandler**==的回调接口。同时==实现==我们==定义的网络请求的服务接口==，**重写网络请求方法**
		- ### 3、当代理类对象方法调用时，通过InvocationHandler. invoke方法进行回调
		- ### 4、所以我们通过retrofit.create拿到代理类对象时，去调用网络请求方法，都会调用到invoke方法进行处理。
	- # step3、代理类对象执行网络请求方法返回call，涉及动态代理和[[call适配过程]]
		- ### 1、代理类对象触发网络请求，会回调create的动态代理中invoke方法。返回一个ExecutorCallBackCall对象，进行网络请求用
			- Call<SharedListBean> sharedListCall = sharedListService.getSharedList(2,1);
		- ### 2、通过loadServiceMethod根据每个Method创建ServiceMethod实例调用其invoke方法。
		- ### 3、实际调用ServiceMethod子类 CallAdapted的Adapt方法 包装okhttpCall返回一个ExecutorCallBackCall(可进行线程切换 切换到主线程)
- ## 4、[[retrofit数据转变过程]]
	- ### 1、当使用[[#red]]==**代理类对象**==调用请求方法时，会返回ExecutorCallBackCall
	- ### 2、 ExecutorCallBackCall.enqueue,实际调用的是OkhttpCall的enqueue，
	- ### 3、OkhttpCall里会利用OkhttpClient创建一个 真正的Okhttp的realCall。进行网络请求
	- ### 4、网络请求回来，parseResponse解析Okhttp的数据
	- ### 5、寻找添加的ResponseBodyConverter响应体转换器，进行解析转换，一般是GsonConverterFactory返回的
- ## 5、[[CallAdapter.adapt方法作用]]
	- ## callAdapter是对网络请求事件的一次包装，封装成一个指定类型的东西，
	- 1、默认返回了一个对okhttpCall的封装的ExecutorCallBackCall。
	- 2、例如如果是rx实现那么就将OkhttpCall 转换成observable对象 可观察的对象，这样其他地方就可以进行订阅这个可监听事件。
- ## 6、ServiceMethod是什么
	- 1、ServiceMethod是用来存储[[#red]]==**一次网络请求的基本信息的**==，比如Host、URL、请求方法、注解上的参数等等
	- 2、一个接口的method调用，都会创建自己的ServiceMethod，这就意味着ServiceMethod是属于函数的
	- 3、[[#red]]==**同时ServiceMethod还会存储用来适配OkHttpCall对象的CallAdpater。**==比如是适配Call的还是适配Obserable的
	- ### 参考
	  collapsed:: true
		- ServiceMethod的build方法会解读传入的Method，
		- 首先ServiceMethod会在CallAdpaterFactory列表中寻找合适的CallAdapter来包装OkHttpCall对象，这一步主要是根据Method的返回参数来匹配的
		- 比如如果方法的返回参数是Call对象，那么ServiceMethod就会使用默认的CallAdpaterFactory来生成CallAdpater，
		- 而如果返回对象是RxJava的Obserable对象，则会使用RxJavaCallAdapterFactory提供的CallAdpater。