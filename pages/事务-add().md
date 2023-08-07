# 概念使用
	- 就是通过事务往容器里添加Fragment。
	- 再使用[[事务-hide-show]] 来决定显示哪个Fragment，隐藏哪个Fragment
		- 在源码中就是对应Fragment中View的显示和隐藏。
- # 使用场景
	- 当Fragment不可见时，如果你==**要保留Fragment中的数据以及View的显示状态，那么可以使用add操作**==，后续中针对不同的状态隐藏和显示不同的Fragment。
- # 优缺点
	- 优点：快，就是Fragment中View的显示和隐藏
	- 缺点：内存中保留的数据太多，容易导致造成OOM的风险。