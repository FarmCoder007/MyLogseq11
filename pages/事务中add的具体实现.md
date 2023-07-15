- # 概况
	- 修改fragmentManager 和   布局ID ，构造成 Op操作对象，设置状态信息，然后添加到操作列表里。
- # 代码
	- ```java
	  @Override
	  public FragmentTransaction add(int containerViewId, Fragment fragment) {
	      doAddOp(containerViewId, fragment, null, OP_ADD);
	      return this;
	  }
	  @Override
	  public FragmentTransaction add(int containerViewId, Fragment fragment, String tag) {
	      doAddOp(containerViewId, fragment, tag, OP_ADD);
	      return this;
	  }
	  ```
	- doAddOp：[[doAddOp]]