title:: CallAdapter.adapt方法作用

- ## callAdapter是对网络请求事件的一次包装，他的作用就是将我们的这个请求封装成一个指定类型的东西，
- 1、默认返回了一个对okhttpCall的封装的ExecutorCallBackCall。
- 2、例如如果是rx实现那么就将OkhttpCall 转换成observable对象 可观察的对象，这样其他地方就可以进行订阅这个可监听事件。
-
-