- FragmentPagerAdapter 与 FragmentStatePagerAdapter 新增了含有 `behavior`
  collapsed:: true
	- 如果 behavior 的值为 `BEHAVIOR_SET_USER_VISIBLE_HINT`，那么当 Fragment 对用户的可见状态发生改变时，`setUserVisibleHint` 方法会被调用。
	- 如果 behavior 的值为 `BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT` ，那么当前选中的 Fragment 在 `Lifecycle.State#RESUMED` 状态 ，其他不可见的 Fragment 会被限制在 `Lifecycle.State#STARTED` 状态
- **viewPager+Fragment，**Androidx下使用时只需要在onResume 下处理懒加载