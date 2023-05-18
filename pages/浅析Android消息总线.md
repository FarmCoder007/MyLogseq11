- 我们常用的消息总线，包括EventBus，RxBus，以及本次侧重为大家介绍的LiveEventBus，接下来给大家介绍在这几种消息总线在设计思想和使用方式的异同
- ## 设计思想
	- 消息总线的设计思想都是观察者模式（订阅/发布模式）
	  collapsed:: true
		- ![image.png](../assets/image_1684421306561_0.png)
	- 订阅发布模式定义了一种“一对多”的依赖关系，让多个订阅者对象同时监听某一个主题对象。这个主题对象在自身状态变化时，会通知所有订阅者对象，使它们能够自动更新自己的状态。
- ## 普通用法
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
		- ##