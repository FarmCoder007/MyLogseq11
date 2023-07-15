- ```java
  FragmentManager mFragmentManager = getSupportFragmentManager();
  mFragmentManager.beginTransaction()
      .replace(R.id.fl_content, fragment)
      .commitAllowingStateLoss();
  ```
- ## 1、用户代码：getSupportFragmentManager();
	- 通过FragmentActivity持有的FragmentController,[[#red]]==**获取FragmentManagerImpl**==
- ## 2、用户代码：fragmentManager.beginTransaction();
	- [[#red]]==**返回 事务FragmentTransaction**== 子类 实例 new[[#red]]==**BackStackRecord**==(this); （操作事务的真正实现）
- ## 3、用户代码：fragmentTransaction.add()
	- [[#red]]==**以及其它操作都是以op的方式存放在mOps操作列表中**==
	- > 把操作都放一个集合里，最终commit一起执行
- ## 4、用户代码：fragmentTransaction.commit();
	- 1、将每个事务加入FragmentManagerImpl队列中，通过Handler发送执行
	- 2、主线程中执行execPendingActions(); 这个任务
	- 3、把所有的事务操作取出来放入临时变量中
	- 4、移除多余的操作  主要是[[#red]]==**优化比如连续操作**==：add   remove   add add同一个fragment时就是多除操作。
	- 5、expandOps：根据不同命令处理，[[#red]]==**比如将replace 转换成 remove + add**==
	- > 在这之前都是在处理各种标记，还没具体执行事务
	- 6、executeOps ：真正执行事务：将每个操作取出，switch语句  根据命令比如OP_ADD去匹配，调用FragmentManagerImpl执行对应的操作
	- 7、最终执行 moveToState(fragment); 执行相应的Fragment生命周期