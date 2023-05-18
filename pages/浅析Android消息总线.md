- 我们常用的消息总线，包括EventBus，RxBus，以及本次侧重为大家介绍的LiveEventBus，接下来给大家介绍在这几种消息总线在设计思想和使用方式的异同
- ## 设计思想
	- 消息总线的设计思想都是观察者模式（订阅/发布模式）
	  collapsed:: true
		- ![image.png](../assets/image_1684421306561_0.png)
	- 订阅发布模式定义了一种“一对多”的依赖关系，让多个订阅者对象同时监听某一个主题对象。这个主题对象在自身状态变化时，会通知所有订阅者对象，使它们能够自动更新自己的状态。
- ## 普通用法
  collapsed:: true
	- ## EventBus
	  collapsed:: true
		- ### 1.定义消息
			- EventBus需要定义一个Java Object作为消息事件
			  collapsed:: true
				- ```
				  public class MessageEvent {
				  
				      public String msg;
				  
				      public MessageEvent(String msg) {
				          this.msg = msg;
				      }
				  }
				  ```
		- ### 2.订阅和取消订阅
		  collapsed:: true
			- EventBus需要在一个生命周期中订阅和取消订阅消息，然后用注解定义接收消息的方法
				- ```
				  @Override
				  protected void onCreate(Bundle savedInstanceState) {
				      super.onCreate(savedInstanceState);
				      setContentView(R.layout.activity_eventbus_demo);
				      EventBus.getDefault().register(this);
				  }
				  
				  @Override
				  protected void onDestroy() {
				      super.onDestroy();
				      EventBus.getDefault().unregister(this);
				  }
				  
				  @Subscribe(threadMode = ThreadMode.MAIN)
				  public void onMessageEvent(MessageEvent event) {
				      Toast.makeText(this, "receive massage: " + event.msg, Toast.LENGTH_SHORT).show();
				  }
				  ```
		- ### 3.发送消息
		  collapsed:: true
			- ```
			  public void sendMsg(View v) {
			      EventBus.getDefault().post(new MessageEvent("msg from event bus"));
			  }
			  ```
	- ## RxBus
		- ## 2.消息的订阅和取消订阅
		  collapsed:: true
			- ```
			    private Subscription mIMKickOffSubscription;  
			  @Override
			  protected void onCreate(Bundle savedInstanceState) {
			      super.onCreate(savedInstanceState);
			      setContentView(R.layout.activity_eventbus_demo);
			         mIMKickOffSubscription = RxDataManager.getBus().observeEvents(IMKickOff.class) 
			                  .observeOn(AndroidSchedulers.mainThread()) //订阅者消费的线程
			                  .subscribe(new RxWubaSubsriber<IMKickOff>() {
			                      @Override
			                      public void onNext(IMKickOff imKickOff) {
			                          IMLoginManager.getInstance().showLoginIMDialog(getActivity());
			                      }
			                  });
			  
			  }
			  
			  @Override
			  protected void onDestroy() {
			      super.onDestroy();
			          RxUtil.safeUnsubscribe(mIMKickOffSubscription);
			  }
			  ```
		- ## 3发送消息
		  collapsed:: true
			- ```
			  public void sendMsg(View v) {
			                      RxDataManager.getBus().post(new IMKickOff(true));
			  }
			  ```
		- ## RxBus原理
		  collapsed:: true
			- 在RxJava中有个Subject类，它继承Observable类，同时实现了Observer接口，因此Subject可以同时担当订阅者和被订阅者的角色，我们使用Subject的子类PublishSubject来创建一个Subject对象（PublishSubject只有被订阅后才会把接收到的事件立刻发送给订阅者），在需要接收事件的地方，订阅该Subject对象，之后如果Subject对象接收到事件，则会发射给该订阅者，此时Subject对象充当被订阅者的角色。
			- 完成了订阅，在需要发送事件的地方将事件发送给之前被订阅的Subject对象，则此时Subject对象作为订阅者接收事件，然后会立刻将事件转发给订阅该Subject对象的订阅者，以便订阅者处理相应事件，到这里就完成了事件的发送和处理。
			- 最后就是取消订阅的操作，RxJava中，订阅操作会返回一个Subscription对象，以便在合适的时机取消订阅，防止内存泄漏，如果一个类产生多个Subscription对象，我们可以用一个CompositeSubscription存储起来，以进行批量的取消订阅。
			- RxBus有很多的实现
			- 基于RxJava1的RxBus实现：
				- ```
				  public final class RxBus {
				  
				      private final Subject<Object, Object> bus;
				  
				      private RxBus() {
				          bus = new SerializedSubject<>(PublishSubject.create());
				      }
				  
				      private static class SingletonHolder {
				          private static final RxBus defaultRxBus = new RxBus();
				      }
				  
				      public static RxBus getInstance() {
				          return SingletonHolder.defaultRxBus;
				      }
				  
				      /*
				       * 发送
				       */
				      public void post(Object o) {
				          bus.onNext(o);
				      }
				  
				      /*
				       * 是否有Observable订阅
				       */
				      public boolean hasObservable() {
				          return bus.hasObservers();
				      }
				  
				      /*
				       * 转换为特定类型的Obserbale
				       */
				      public <T> Observable<T> toObservable(Class<T> type) {
				          return bus.ofType(type);
				      }
				  }
				  
				  ```
- ## 引入LiveDataBus
	- LiveData是一个可以观察的数据持有类，它可以感知并遵循Activity，Fragment或Service等组件的生命周期。正是由于LiveData对组件生命周期可感知特点，因此可以做到仅在组件处于生命周期的激活状态时才更新UI数据。
	- LiveData需要一个观察者对象，一般是Observer类的具体实现。当观察者的生命周期处于STARTED或RESUMED状态时，LiveData会通知观察者数据变化；
	- 在观察者处于其他状态时，即使LiveData的数据变化了，也不会通知。
	- LiveData的优点
	- UI和实时数据保持一致
	- 避免内存泄漏
	- 不会再产生由于Activity处于stop状态而引起的崩溃