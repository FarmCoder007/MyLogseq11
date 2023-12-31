# Android内存泄漏常见场景以及解决方案#card
	- ## 1、资源性对象未关闭
		- 对于资源性对象不再使用时，应该立即调用它的close()函数，将其关闭，然后再置为null。
		- 例如[[#green]]==**Bitmap,io等资源**==未关闭会造成内存泄漏，此时我们应该在[[#green]]==**Activity销毁时及时关闭。**==
	- ## 2、注册对象未注销
		- 例如[[#red]]==**BraodcastReceiver、EventBus未注销**==造成的内存泄漏，我们应该在Activity销毁时及时注销。
	- ## 3、类的静态变量持有大数据对象
		- 尽量避免使用静态变量存储数据，特别是大数据对象，建议使用数据库存储。
	- ## 4、单例造成的内存泄漏
		- 优先使用Application的Context，如需使用Activity的Context，可以在传入Context时使用弱引用进行封装，然后，在使用到的地方从弱引用中获取Context，如果获取不到，则直接return即可。
	- ## 6、[[#green]]==**Handler临时性内存泄漏**==
	  collapsed:: true
		- Message发出之后存储在MessageQueue中，在Message中存在一个target，它是Handler的一个引
		  用，Message在Queue中存在的时间过长，就会导致Handler无法被回收。如果Handler是非静态的，
		  则会导致Activity或者Service不会被回收。并且消息队列是在一个Looper线程中不断地轮询处理消息，
		  当这个Activity退出时，消息队列中还有未处理的消息或者正在处理的消息，并且消息队列中的Message持有Handler实例的引用，Handler又持有Activity的引用，所以导致该Activity的内存资源无法及时回收，引发内存泄漏。解决方案如下所示：
		- 1、使用一个静态Handler内部类，然后对Handler持有的对象（一般是Activity）使用弱引用，这
		  样在回收时，也可以回收Handler持有的对象。
		- 2、在Activity的Destroy或者Stop时，应该移除消息队列中的消息，避免Looper线程的消息队列中
		  有待处理的消息需要处理。
		- 需要注意的是，AsyncTask内部也是Handler机制，同样存在内存泄漏风险，但其一般是临时性的。对于类似AsyncTask或是线程造成的内存泄漏，我们也可以将AsyncTask和Runnable类独立出来或者使用静态内部类。
	- ## 7、==**容器中的对象没清理**==造成的内存泄漏
		- 在退出程序之前，将集合里的东西clear，然后置为null，再退出程序
	- ## 8、[[#green]]==**WebView**==
		- WebView都存在内存泄漏的问题，在应用中只要使用一次WebView，内存就不会被释放掉。我们可以为[[#green]]==**WebView开启一个独立的进程，使用AIDL与应用的主进程进行通信**==，WebView所在的进程可以根据业务的需要选择合适的时机进行销毁，达到正常释放内存的目的。
- ##### 9、使用ListView时造成的内存泄漏
	- 在构造Adapter时，使用缓存的convertView。
- ##### 5、非静态内部类的静态实例
	- 该实例的生命周期和应用一样长，这就导致该静态实例一直持有该Activity的引用，Activity的内存资源不能正常回收。此时，我们可以将该内部类设为静态内部类或将该内部类抽取出来封装成一个单例，如果需要使用Context，尽量使用Application Context，如果需要使用Activity Context，就记得用完后置空让GC可以回收，否则还是会内存泄漏。