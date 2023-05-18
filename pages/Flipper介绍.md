- # 介绍
  collapsed:: true
	- Flipper是一个基于Electron开发的桌面端应用，支持Window/Linux/Mac等多平台，通过桌面插件和其他插件可以扩展其功能。
		- 桌面插件：可以通过公共插件市场、私有插件市场、内置等多种方式安装到Flipper应用中，它又划分为client/device两种类型。
		- 其他插件：支持Android、iOS、React-Native、Browser/Node等各平台，需要在各端应用中嵌入其SDK和插件相关代码。
	- 桌面插件与设备插件之间通过TCP链接传输数据，最后统一由桌面插件展示数据，处理交互，以React-Native插件交互为例，其整体结构如下：
		- ![image.png](../assets/image_1684428470935_0.png)
- # Flipper桌面插件
	- 它指的是在Flipper桌面客户端上安装使用的插件，使用typescript语言编写，一般情况下通过与Flipper其他插件交互获取数据，然后在Flipper桌面客户端上进行展示。
- #