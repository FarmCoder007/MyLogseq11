## 1、首先add和replace都可以在Activity上添加Fragment
- # 使用上的区别
  collapsed:: true
	- ## 首次添加
	  collapsed:: true
		- 1、当前Activity同一个容器上没有Fragment。第一次添加一个Fragment时add和replace效果是一样的。
		- [[#red]]==**生命周期变化也一样**==
			- onAttach->onCreate->onCreateView->onActivityCreated->onStart-onResume,
			- Fragment所依附的Activity销毁时，执行onPause->onStop->onDestoryView->onDestory->onDetach
	- ## 已有再添加
		- ## Add操作
			- 如果FragmentA时通过add添加的，在将FragmentA替换为FragmentB时，可以通过hide FragmentA，add FragmentB  show FragmentB
		- ## replace操作
			- 如果FragmentA通过replace操作添加的，在将FragmentA替换为FragmentB时，使用replace替换
- # 使用场景上的区别：隐藏后是否保存状态
  collapsed:: true
	- # [[事务-add()]]
	- # [[事务-replace]]
-
- # add 和 replace 的区别-面试总结
	- ## 1、首先add和replace都可以在Activity上添加Fragment
	- ## 2、add添加Fragment后，再通过hide show 决定显示哪个
		- 隐藏的Fragment会保留数据和view状态。
		- 优点显示隐藏快，缺点，保留太多数据会造成oom
	- ## 3、replace本质是remove和add操作，旧的 Fragment走生命周期销毁流程，新传递的Fragment走生命周期创建流程。
		- Fragment不可见时，不会保留数据和view状态
		- 优点：节省内存，不需要的数据及时释放点
		- 缺点：频繁创建Fragment，会频繁走创建销毁，造成性能开销
	- ## 4、真正添加上也有区别
		- ## 1、该id第一次添加Fragment，replace操作和add操作一样
			- 两个方法对应生命周期变化也一致：onAttach->onCreate->onCreateView->onActivityCreated->onStart-onResume,
			- Fragment所依附的Activity销毁时，执行onPause->onStop->onDestoryView->onDestory->onDetach
		- ## 2、该id已有Fragment
			- 1、replace传递的Fragment实例和已存在的Fragment实例一样，replace无效果。
			- 2、不一样的话
				- replace操作会转换为 remove和add操作，即删除旧的Fragment,添加新的Fragment。
				- 旧的Fragment执行 onPause->onStop->onDestoryView->onDestory->onDetach
				- 新的Fragment执行 onAttach->onCreate->onCreateView->onActivityCreated->onStart-onResume