- # 一、概念
	- **Framework层的窗口管理服务，它运行在System_server进程**,
- # 二、职责
	- WMS主要管理WIndow的，
	- 是[[#red]]==**管理Android系统中的所有的Window创建、更新和删除，显示顺序等不管实际绘制**==。窗口管理服务，继承IWindowManager.Stub，Binder服务端，因此WM与WMS的交互也是一个IPC的过程
- 负责
	- Z-ordered的维护函数（z轴window展示顺序维护）
	- 输入法管理
	- AddWindow/RemoveWindow
	- Layerout
	- Token管理，AppToken
	- 活动窗口管理（FocusWindow）
	- 活动应用管理（FocusAPP）
	- 转场动画
	- 系统消息收集线程
	- 系统消息分发线程