- ### 1、整个retrofit 采用的时[[#red]]==**外观模式**==。统一的调用创建网络请求接口实例和网络请求参数配置的方法
- ## 2、Retrofit 创建时的callFactory，使用[[#red]]==**工厂方法设计模式**==
- ## 3、Retrofit 实例使用[[#red]]==**建造者模式通过Builder类构建**==
- ## 4、retrofit里面使用了[[#red]]==**动态代理**==来创建网络请求接口实例
- ## 5、ExecuteCallBack 使用[[#red]]==**装饰者模式来封装callbackExecutor**==，用于完成线程的切换
- ## 6、CallAdapter采用了[[#red]]==**适配器模式**== 为创建访问Call接口提供服务
- ## 7、ExecutorCallbackCall 使用[[#red]]==**静态代理(委托) 代理了Call**==进行网络请求
- ## 8、使用了**策略模式**对serviceMethod对象进行网络请求参数配置