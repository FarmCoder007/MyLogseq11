- # 一、概述
	- ## BackStackRecord
		- 是对 [[#red]]==Fragment 进行操作的事务的真正实现==，[[#red]]==所以每个这个都相当于一个事务==
		- 也是 FragmentManager 中的回退栈的实现：[[#red]]==和FragmentManagerImpl相互持有==
		- ```java
		  final class BackStackRecord extends FragmentTransaction implements
		    FragmentManager.BackStackEntry, FragmentManagerImpl.OpGenerator {...}
		  ```
	- ## FragmentImpl.beginTransaction中返回的 BackStackRecord :
		- ```java
		  @Override
		  public FragmentTransaction beginTransaction() {
		  	return new BackStackRecord(this);
		  }
		  ```
- # 二、它的关键成员：
	- ```java
	  
	  final FragmentManagerImpl mManager;
	  //Op 可选的状态值
	  static final int OP_NULL = 0;
	  static final int OP_ADD = 1;
	  static final int OP_REPLACE = 2;
	  static final int OP_REMOVE = 3;
	  static final int OP_HIDE = 4;
	  static final int OP_SHOW = 5;
	  static final int OP_DETACH = 6;
	  static final int OP_ATTACH = 7;
	  ArrayList<Op> mOps = new ArrayList<>();
	  static final class Op {
	      int cmd; //状态
	      Fragment fragment;
	      int enterAnim;
	      int exitAnim;
	      int popEnterAnim;
	      int popExitAnim;
	  }
	  int mIndex = -1; //栈中最后一个元素的索引
	  ```
	- 可以看到 Op 就是添加了状态和动画信息的 Fragment，
	- mOps 就是栈中所有的 Fragment