- # 一、概述
	- 在Android 系统中， DVM (Dalvik 虚拟机）和 ART 应用程序进程以及运行系统的关
	  键服务的 SystemServer 进程都是由 Zygote 进程来创建的，我们也将它称为孵化器。它通过
	  fock (复制进程）的形式来创建应用程 进程和 SystemServer 进程，由于 Zygote 进程在启
	  动时会创建 DVM 或者 ART ，因此通过 fock 而创建的应用程序进程和 System Server 进程可
	  内部获取 DVM 或者 ART 例副本