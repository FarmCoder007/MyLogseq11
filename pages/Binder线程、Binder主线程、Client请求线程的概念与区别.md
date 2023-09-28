# 1、Binder 线程：
	- Binder 线程是指 ==**Android 系统**==中[[#red]]==**专门用于处理进程间通信**==（IPC）的线程。是执行Binder服务的载体，只对于服务端才有意义，对请求端来说，是不需要考虑Binder线程的
- # 2、Binder 主线程
	- ProcessState::self()->startThreadPool()是新建一个Binder主线程，而PCThreadState::self()->joinThreadPool()是将当前线程变成Binder主线程。
- # 3、普通线程：
	- 普通线程是指在==**应用程序中创建的常规线程**==
	- 普通Client的binder请求线程，比如我们APP的主线程，在startActivity请求AMS的时候，APP的主线程成其实就是Binder请求线程，在进行Binder通信的过程中，Client的Binder请求线程会一直阻塞，知道Service处理完毕返回处理结果
- # Binder线程与Binder主线程的区别是
	- [[#red]]==**线程是否可以终止Loop**==，主线程不会退出一直循环loop
-