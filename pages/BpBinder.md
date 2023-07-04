- ![image.png](../assets/image_1688359965036_0.png)
- # 左侧相当于客户端，右侧是服务端（里边有SM对象）
	- 客户端想要拿到SM对象，java里是直接new。但是跨进程是不能直接new的。
		- 客户端想要和服务端通信，拿到里边的SM对象(相当于BBinder就是SM对象)是没办法直接拿到的
	- 服务端将BBinder（相当于SM）注册到内核层的Binder驱动中
	- 客户端通过binder驱动，拿到的BpBinder 相当于 BBinder的代理对象。
		- 可以通过Binder，拿到BpBinder（相当于BBinder的一个代理对象，代理SM）
- # Binder框架类图：涉及的方法
  collapsed:: true
	- ![image.png](../assets/image_1688360884472_0.png)
- # Java层怎么获取BBinder的
  collapsed:: true
	- ![image.png](../assets/image_1688361038513_0.png)
- # [[Native层怎么获取BpBinder的]]
	-