title:: docusaurus.config.js

- # 一、介绍
	- 他是docusaurus的配置文件。里面包含了这几个部分：
		- 1 站点元数据
		  2 部署配置
		  3 主题,插件和预设配置
	- 4 自定义配置
- # 二、配置含义
	- 必须字段：
		- title：string类型，定义网站的标题
		- baseUrl：string类型，定义站点的根路径，没有路径的 URL就设置baseUrl: '/',
		              /metro/ 是 /metro/baseUrl 的 baseUrl
		- url：string类型，定义网站域名
		- favicon：string类型，图标文件的url
	- 可选字段：
	  1 noIndex: true：会在页面添加，阻止搜索引擎搜索你的网站