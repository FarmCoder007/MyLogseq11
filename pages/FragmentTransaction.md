-
- # 一、操作方法FragmentTransaction 定义了一系列对 Fragment 的操作方法：
	- [[#red]]==**hide 和 show不会走生命周期方法**==
	- ```java
	  //它会调用 add(int, Fragment, String)，其中第一个参数传的是 0
	  public abstract FragmentTransaction add(Fragment fragment, String tag);
	  
	  //它会调用 add(int, Fragment, String)，其中第三个参数是 null
	  public abstract FragmentTransaction add(@IdRes int containerViewId,Fragment fragment);
	  
	  //添加一个 Fragment 给 Activity 的最终实现
	  //第一个参数表示 Fragment 要放置的布局 id
	  //第二个参数表示要添加的 Fragment，【注意】一个 Fragment 只能添加一次
	  //第三个参数选填，可以给 Fragment 设置一个 tag，后续可以使用这个 tag 查询它
	  public abstract FragmentTransaction add(@IdRes int containerViewId,
	  													Fragment fragment, @Nullable String tag);
	  
	  //调用 replace(int, Fragment, String)，第三个参数传的是 null
	  public abstract FragmentTransaction replace(@IdRes int containerViewId,Fragment fragment);
	  
	  //替换宿主中一个已经存在的 fragment
	  //这一个方法等价于先调用 remove(), 再调用 add()
	  public abstract FragmentTransaction replace(@IdRes int containerViewId, Fragment fragment,
	  																@Nullable String tag);
	  //移除一个已经存在的 fragment
	  //如果之前添加到宿主上，那它的布局也会被移除
	  public abstract FragmentTransaction remove(Fragment fragment);
	  
	  
	  //隐藏一个已存的 fragment
	  //其实就是将添加到宿主上的布局隐藏
	  public abstract FragmentTransaction hide(Fragment fragment);
	  
	  
	  //显示前面隐藏的 fragment，这只适用于之前添加到宿主上的 fragment
	  public abstract FragmentTransaction show(Fragment fragment);
	  
	  
	  //将指定的 fragment 将布局上解除
	  //当调用这个方法时，fragment 的布局已经销毁了
	  public abstract FragmentTransaction detach(Fragment fragment);
	  
	  
	  //当前面解除一个 fragment 的布局绑定后，调用这个方法可以重新绑定
	  //这将导致该 fragment 的布局重建，然后添加、展示到界面上
	  public abstract FragmentTransaction attach(Fragment fragment);
	  ```
- # 二、示例
	- 对 fragment 的操作基本就这几步，我们知道，要完成对 fragment 的操作，最后还需要提交一下：
	- ```java
	  mFragmentManager.beginTransaction()
	  .replace(R.id.fl_child, getChildFragment())
	  // .commit()
	  .commitAllowingStateLoss();
	  ```
-