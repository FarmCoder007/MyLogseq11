# add,remove()
	- 使用add()加入fragment时将触发onAttach(),使用attach()不会触发onAttach()
- # replace()
	- 使用replace()替换后会将之前的fragment的view从viewtree中删除，不保存Fragment数据和view状态
- # hide 和 show
	- 使用hide()方法只是隐藏了fragment的view并没有将view从viewtree中删除,随后可用show()方法将view设置为显示
	- [[#red]]==**hide和show 不会执行 Fragment相应的生命周期**==
-
- 而使用detach()会将view从viewtree中删除,和remove()不同,此时fragment的状态依然保持着,在使用attach()时会再次调用onCreateView()来重绘视图,注意使用detach()后fragment.isAdded()方法将返回false,在使用attach()还原fragment后isAdded()会依然返回false(需要再次确认)
- 执行detach()和replace()后要还原视图的话, 可以在相应的fragment中保持相应的view,并在onCreateView()方法中通过view的parent的removeView()方法将view和parent的关联删除后返回