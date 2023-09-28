## 1：初始化callFactory 为OkhttpClient。[[#red]]==**创建RealCall用**==
	- 后边创建Okhttp realCall用
- ## 2：用handler封装的Executor[[#red]]==**做线程切换用**==
- ## 3：设置默认CallAdapterFactory，提供默认的CallAdapter转换器
	- > 这里设置的默认的CallAdapter是 defaultCallAdapterFactory，这个类会被ServiceMethod调用，比如 代理类执行代码的时候会调用到动态代理的invoke 方法处，通过这个defaultCallAdapterFactory的Adapt 方法将 OkhttpCall 封装成 ExecutorCallBackCall进行返回
	- [[#red]]==**默认的Adapter把okhttpcall 适配成Retrofit call（叫ExecutorCallBackCall）**==
- ## 4：初始化转换器converterFactories，[[#red]]==**对请求和返回数据进行转换处理**==