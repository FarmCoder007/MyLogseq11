- # 一、概述
	- 在Android 系统中， DVM (Dalvik 虚拟机）和 ART、应用程序进程、运行系统的关键服务的SystemServer 进程都是由 Zygote 进程来创建的，我们也将它称为孵化器。它通过fock (复制进程）的形式来创建应用程序进程和 SystemServer 进程，由于 `Zygote 进程在启动时会创建 DVM 或者 ART` ，因此通过 fock 而创建的应用程序进程和 System Server 进程可内部获取 DVM 或者 ART 例副本
-
- Android 5.0 开始， Android 开始 64 位程序， Zygote 也就有了 32 位和 64 位的区
  别，所以在这里用 ro.zygote 属性来控制使用不同的 Zygote 启动脚本，从而也就启动了不同
  版本的 Zygote 进程， ro zygote 属性的取值有以下 种：