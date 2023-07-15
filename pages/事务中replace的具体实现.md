- ## 代码
	- ```java
	  @Override
	  public FragmentTransaction replace(int containerViewId, Fragment fragment)
	  {
	  	return replace(containerViewId, fragment, null);
	  }
	  @Override
	  public FragmentTransaction replace(int containerViewId, Fragment fragment, String tag) {
	      if (containerViewId == 0) {
	          throw new IllegalArgumentException("Must use non-zero containerViewId");
	      }
	      doAddOp(containerViewId, fragment, tag, OP_REPLACE);
	      return this;
	  }
	  ```
- 也是调用上面刚提到的[[doAddOp]] ，不同之处在于第四个参数为 OP_REPLACE ，看来之前小看了这个状态值！