# 1、SystemServiceManager
	- 1、位于Java层的Android [[#red]]==**Framework**==中。
	- 2、它是一个[[#red]]==**系统服务管理器专门用于启动、绑定和管理各种系统服务，例如：AMS等创建**==
	- 3、SystemServiceManager负责维护系统服务的生命周期和跨进程通信。
- # 2、ServiceManager
	- 1、也位于Java层的Android Framework中。
	- 2、它是一个[[#red]]==**跨进程通信机制的管理器**==，[[#red]]==**负责管理和提供进程间通信（IPC）机制中的Binder对象**==。
	- 3、ServiceManager充当了客户端与系统服务之间的中介，它允许应用程序通过[[#red]]==**系统服务的名称获取对应的Binder对象，并进行进程间通信**==。
- SystemServiceManager负责创建管理系统服务，比如AMS的创建，
- ServiceManager是binder机制大管家，负责管理和提供系统服务对应的binder对象。
- 系统服务由SystemServiceManager进行管理，而ServiceManager则提供了访问系统服务的接口。