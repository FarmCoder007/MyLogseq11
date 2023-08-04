- 1、需要提供BaseApplication，放入 最底层 base层，提供获取Context 等通用能力
  collapsed:: true
	- ![image.png](../assets/image_1690941304869_0.png)
- 2、在app层（最上层），新建子类去继承，提供实现，[[#red]]==**清单文件注册**==
  collapsed:: true
	- ![image.png](../assets/image_1690941324963_0.png)
	- 注册
		- ![image.png](../assets/image_1690941343938_0.png)
-
- # 好处
	- ![image.png](../assets/image_1690941428229_0.png)
	- Application Context 不会有内存泄漏