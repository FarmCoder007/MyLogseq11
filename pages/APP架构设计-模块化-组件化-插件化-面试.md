## 1、插件化，组件化，模块化区别
	- ![image.png](../assets/image_1691461730999_0.png)
	- ## [[ 模块化]]
		- 按照业务逻辑，将功能属于同一业务的代码整合到一起，划分成一个一个模块，[[#red]]==**网络模板，base模块，Common模块**==
	- ## [[组件化]]原包
		- 有些组件或基础库在多个模块中都有调用，把这些耦合度高的提炼出来，造成可复用组件，
		- 组件间互相不依赖，抽离接口下沉到Common层，组件可复用，独立开发与调试，类似SDK类型
	- ## [[插件化]]以一个app模式存在
		- 1、将应用程序中的==**功能模块**==，==**变成免安装独立apk插件**==(可独立开发，更新)
		- 2、应用程序运行时可以[[#red]]==**动态加载和卸载插件APK**==
		-
		-
- ## 插件化和组件化区别#card
	- ## 模块化
		- [[#red]]==**按照业务逻辑，将功能属于同一业务的代码整合到一起**==，划分成一个一个模块，[[#red]]==**网络模板，base模块，Common模块**==
	- ## 组件化
		- 1、逻辑上：基础库在多个模块中都有调用，把这些耦合度高的提炼出来，造成可复用底层基础组件，==类似SDK类型==
		- 2、每个模块都是一个组件,最终打包组件是合成一个apk的
	- ## 插件化
		- 1、将应用程序中的==**功能模块**==，==**变成免安装独立apk插件**==(可独立开发，更新)
		- 2、每个模块都是一个插件APK，最终打包的时候 宿主apk和插件apk分开打包。
		-
- ![image.png](../assets/image_1691938032711_0.png)
- ![image.png](../assets/image_1692620997105_0.png)