- # 一、简介
	- 插件可以封装一系列任务，例如 编译，测试，打包等
- # 二、插件分类
	- ## 脚本插件：如xxx.gradle,脚本模块化基础
		- 脚本插件通常是一个脚本，和一个普通的 build.gradle 文件没什么区别
		- 它是脚本模块化的基础。我们可以把复杂的脚本文件，进行分块，分段整理，拆分成一个个职责分明的脚本插件。
		- ### 使用：
			- ```groovy
			  //使用本地插件
			  apply from: './other.gradle'
			  apply from: this.rootProject.file('other.gradle')
			  
			  //使用网络远程插件
			  apply from: 'https://gitee.com/xxx/xx/raw/master/it235.gradle'
			  
			  ```
	- ## 二进制插件：
		- 我们可以通过二进制插件的ID来应用