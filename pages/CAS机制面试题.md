# 1、原子操作
collapsed:: true
	- 不可在拆分的操作
- # 2、[[CAS机制原理]]
  collapsed:: true
- # 3、CAS问题
  collapsed:: true
	- ABA问题
	- 最坏情况下一直循环指令，引起开销问题
	- 只能保证一个共享变量的原子操作
- # 4、[[怎么解决ABA问题？]]
	- 加版本戳，看有没被修改过，获取变量时，会带版本戳，比如JDK提供的实现
		- AtomicMarkableReference
			- 只关心变量有没有被人动过
		- AtomicStampedReference
			- 不仅关心变量有没有被人动过，还会记录动了多少次