title:: docusaurus.config.js

- # 一、介绍
	- 他是docusaurus的配置文件。里面包含了这几个部分：
		- 1 站点元数据
		- 2 部署配置
		- 3 主题,插件和预设配置
		- 4 自定义配置
- # 二、配置含义
	- 必须字段：
		- title：string类型，定义网站的标题
		- baseUrl：string类型，定义站点的根路径，没有路径的 URL就设置baseUrl: '/',
		              /metro/ 是 /metro/baseUrl 的 baseUrl
		- url：string类型，定义网站域名
		- favicon：string类型，图标文件的url
	- 可选字段：
		- noIndex: 
		  collapsed:: true
			- ```
			  <meta name="robots" content="noindex, nofollow">
			  ```
			- true：会在页面添加，阻止搜索引擎搜索你的网站
		- onBrokenLinks
		  collapsed:: true
			- 网站上线了，docusaurus build模式下，就会检测链接是否失效，失效就会抛出错误，错误的等级可以定制：'ignore' | 'log' | 'warn' | 'error' | 'throw'
		- onBrokenMarkdownLinks
		  collapsed:: true
			- 会打印一个警告，让您知道已损坏的markdown链接，错误等级可以定制：'ignore' | 'log' | 'warn' | 'error' | 'throw'
		- onDuplicateRoutes
		  collapsed:: true
			- 执行 yarn start 或 yarn build 之后会对重复路由发生警告，你依然可以调节'ignore' | 'log' | 'warn' | 'error' | 'throw'等级
		- tagline：
		  collapsed:: true
			- 指定网站的标语
		- organizationName
		  collapsed:: true
			- 表示拥有此源码仓库的 GitHub 用户或组织，部署的时候会用到
		- projectName
		  collapsed:: true
			- GitHub 源码仓库的名称
		- githubHost
		  collapsed:: true
			- GitHub 服务器的主机名。如果你使用的是 GitHub 企业版，则会用到此参数
		- themeConfig对象可以定义网站主题分以下三部分：
			- colorMode:
			- navbar控制顶部导航栏，分三部分
				- title: 导航上站点名称
				- logo:
				- items
			- footer:
-