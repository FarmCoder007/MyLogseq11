- ```java
  FragmentManager mFragmentManager = getSupportFragmentManager();
  mFragmentManager.beginTransaction()
      .replace(R.id.fl_content, fragment)
      .commitAllowingStateLoss();
  ```
- ### 1、首先通过getSupportFragmentManager();==**获取FragmentManager实现类FragmentManagerImpl**==
  collapsed:: true
	- 通过FragmentActivity持有的FragmentController,[[#red]]==**获取FragmentManagerImpl**==
- ### 2、然后通过fragmentManager.beginTransaction();[[#red]]==**拿到FragmentTransaction事务的实现类BackStackRecord**==
  collapsed:: true
	- [[#red]]==**返回 事务FragmentTransaction**== 子类 实例 new[[#red]]==**BackStackRecord**==(this); （操作事务的真正实现）
- ### 3、==**当执行**==fragmentTransaction.add()、Replace（）等==**事务操作方法时**==，事务实现类BackStackRecord会==**将目标 Fragment 构造成op操作对象（包装Fragment 和状态信息），存入 Ops操作列表中**==
  collapsed:: true
	- [[#red]]==**以及其它操作都是以op的方式存放在mOps操作列表中**==
	- > 把操作都放一个集合里，最终commit一起执行
- ### 4、最终将[[#red]]==**事务操作列表ops通过commit统一提交，传给FragmentManagerImpl进行处理**==
	- 1、如果是commit 和commitAllowingStateLoss [[#red]]==**异步提交**==，就通过 ==**Handler 发送 Runnable 任务**==将每个事务传给FragmentManagerImpl
	- 2、==**同步**==commitNow 和commitNowAllowingStateLoss ,==**就直接交给**==FragmentManagerImpl处理
- ### 5、FragmentManagerImpl将收到的事务放入临时变量中，进行优化标记，处理 Ops状态
	- 1、[[#red]]==**优化比如连续操作**==：add   remove   add add同一个fragment时就是多除操作。
	- 2、命令分别处理标记：[[#red]]==**比如将replace 转换成 remove + add**==
- ### 6、真正执行事务：executeOps（）方法
	- 将每个操作取出==**，根据OP_ADD等OP命令 去匹配switch语句 ，调用FragmentManagerImpl执行对应的操作**==
- ### 7、最终执行 ==**moveToState(fragment); 根据状态调用 Fragment 对应的生命周期方法**==从而达到Fragment 的添加、布局的替换隐藏等
-