- # 一、概念
	- 管理window的
- # 二、作用
	- window           WM（WMImpl(WMGlobal（）)）            WMS
		- [[#red]]==**window和WMS通信 通过中间的这个WIndowManager**==
		- 由上可知实际通过[[#red]]==WMGlobal中的Session 和WMS通信的==
	- 详细见[[Session]]