title:: observeOn(AndroidSchedulers.mainThread())

- 1、AndroidSchedulers.mainThread()是创建主线程的handler，来切换到主线程的
- [[#red]]==**（订阅前不会切线程，只在拿到回调结果才切线程）**==
- 2、observeOn会创建一个==**中间被观察者持有上游观察者**==
- 3、订阅时  [[#red]]==**创建一个包装观察者持有下游自定义观察者**==
- 4、让上游订阅他包装观察者，包装观察者收到上游回调消息后==**才切线程到主线程**==，然后回调给下游观察者；
-
- # 多次  observeOn调用 只有写在最下一行的才对回调起作用
- # [[ObserveOn()给下边代码分配线程]]