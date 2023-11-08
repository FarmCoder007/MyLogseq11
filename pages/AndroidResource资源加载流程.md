# AndroidResource资源加载流程#Card
	- 1、在ActivityThread的[[#red]]==**handleBindApplication**==方法中
		- [[#red]]==**创建Application实例和 Instrumentation**==
	- 2、创建Application 过程中，会==**先创建Application的Context**==
	- 3、创建上下文中，会加载apk里资源
		- ResourcesManager——>调用到AssetManager—>通过Native方法，[[#red]]==**根据apk路径，读取里边资源生成生成resources.arsc 二进制文件**==（里边记录着资源信息，资源名字：资源路径）
	- > 插件化换肤就可以 传入插件apk路径，去生成插件的二进制文件。用于读取属性 替换属性用