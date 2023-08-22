# 前提，熟悉startActivity的流程
-
- ## 准备工作
	- 1，一个注册在清单的占位activity：StubActivity
	- 2、一个目标activity，偷梁换柱启动这个
- # 思路：比如startActivity简单流程，寻找hook点
  collapsed:: true
	- -> AMS(验证activity是否注册)
		- 这一步不能传未注册的activity（TargetActivity）需要传StubActivity，会被检测到
		- 传注册的activity，AMS检测启动结果一定是res sucess 成功的
		- [[#red]]==**Hook点传个已注册的activity**==
	- -> 回到app进程后（Instrumentation 创建Activity）
		- 这里回到app时，activity信息是StubActivity
		- [[#red]]==**hook点：stubActivity替换回targetActivity**==
	- -> AMS 控制 Instrumentation 执行Activity生命周期
-
- # Hook点选择，这些修饰符，修饰的hook点比较好
  collapsed:: true
	- public
	- static
	- 单例
- # 插件化启动插件里的Activity，动态代理实现hook
	-