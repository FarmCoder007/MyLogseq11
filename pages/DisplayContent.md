- ## 一、用于描述多屏输出相关信息。
- 根据窗口的显示位置将其分组。
	- [[#red]]==**隶属于同一个DisplayContent的窗口将会被显示在同一个屏幕中**==。
	- ==**每一个DisplayContent都对应这唯一ID，在添加窗口时可以通过指定这个ID决定其将被显示在那个屏幕中。**==
- DisplayContent是一个非常具有隔离性的一个概念。处于不同DisplayContent的两个窗口在布局、显示顺序以及动画处理上不会产生任何耦合