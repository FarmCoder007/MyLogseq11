- ## 问题描述：
	- 由于okhttpClient 可使用单例，也可以不使用单例，那么Dipatcher调度器也是在okhttpClient中创建的，会导致[[#red]]==**每创建一个client就创建一个分发器，每个分发器创建一个线程池**==。
	- 直接使用 OkHttpClient 执行请求的场景下，如果创建多个 OkHttpClient 对象，会生成对应数量的线程池，对应着也会生成相应数量的线程，每个请求都在自己的线程池中执行，实际上线程池也就失去了意义
	-
	- ## Retrofit结合RXJava时
		- 同步请求是在RXjava的io线程执行的
		- 异步请求，rxjava io线程里，最终还是okhttp分发器里线程池执行的。创建了多次线程
	- ## 58的做法：
		- 结合RXjava，在rxjava io线程中执行OKhttp的 同步任务。这样就不会使用OKhttp的线程池了。
		- 但是注意rxjava io线程 没有做数量限制。也避免同时大量请求