title:: binding.setUser(user);

- ## 实际走的ActivityMainBindingImpl 中的setUser 方法
	- ```java
	  public void setUser(@Nullable com.example.databindingdemo_20210117.User User) {
	    updateRegistration(0, User);
	    this.mUser = User;
	    synchronized(this) {
	      mDirtyFlags |= 0x1L;
	    }
	    notifyPropertyChanged(BR.user);
	    super.requestRebind();
	  }
	  
	  ```
- ## 第一块，[[ViewDataBinding-updateRegistration更新注册监听]]
- ## 第二块，[[notifyPropertyChanged]]，通知属性改变
-