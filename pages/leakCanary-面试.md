# 1、找出有泄漏嫌疑的对象
- # 2、确诊，借助haha框架可达性分析
- # 一、只需要添加一行依赖，不需要 初始化的原因：
	- ContentProvider#onCreate 会在 Application#onCreate 之前先执⾏ ， 在2.0在ContentProvider 的  onCreate 中就可以进⾏初始化了。 LeakCanary 2.0 利⽤了这个 原理，所以不需要我们⼿动进⾏初始化
- # 2、[[LeakCanary原理]]