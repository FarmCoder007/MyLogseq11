- ## 应用场景：获取Map<String , T>添加key集合
- ## 一、简介
	- The stateful transformation require memorizing some state about the instructions that have been visited before the current one. This requires storing state inside the method adapter 有状态转换需要记住一些在当前指令之前访问过的指令状态，这需要在方法适配器中存储状态
- ## 二、官方示例
	- 1、[删除指令](https://www.yuque.com/mikaelzero/asm/dtooxk)
	  collapsed:: true
		- 移除ICONST_0 IADD。例如，int d = c + 0;与int d = c;两者效果是一样的，所以+ 0的部分可以删除掉
	- 2、[删除指令: ](https://www.yuque.com/mikaelzero/asm/dtooxk)
	  collapsed:: true
		- 移除ALOAD_0 ALOAD_0 GETFIELD PUTFIELD。例如，this.val = this.val;，将字段的值赋值给字段本身，无实质意义
	- 为何说stateless transformation实现起来比较容易，而stateful transformation会实现起来比较困难呢？
	  collapsed:: true
		- stateless transformation不依赖于其他Instruction，只需关注自身，因此实现起来比较简单
		- stateful transformation 依赖其他一条或多条Instruction同时判断，多个指令是一个组合，不能轻易拆散。
	- 实现步骤一般分为三步：
		- 1、将问题转换成Instruciton指令，然后对多个指令组合的特征或遵循的模式进行总结；
		- 2、根据总结的特征或模式对指令进行识别，在识别的过程中，每一条Instruction的加入都会引起原有状态state的变化，这就是stateful的含义；
		- 3、识别成功之后，要对Class文件进行转换，这就对应transformation部分，无非就是对Instruction的内容进行增删改等操作；
		- 如何根据识别记录状态（state）变化呢，这里就要用到state machine(状态机)
- ## 三、状态机state machine
	-