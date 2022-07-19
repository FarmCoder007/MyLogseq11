- ## 一、简介
	- [Docusaurus](https://www.docusaurus.cn/docs/installation)是一个静态站点生成器。它构建了一个具有快速客户端导航的单页应用程序，利用React的全部功能使您的站点具有交互性。它提供开箱即用的文档功能，但可用于创建任何类型的网站（个人网站、产品、博客、营销登录页面等）
- ## 二、配置详解
	- ### [[docusaurus.config.js]]
		-
- ## 三、项目目录
  collapsed:: true
	- ```
	  ├── README.md
	  ├── babel.config.js
	  ├── blog                             // 博客文章目录
	  │   ├── 2019-05-28-hola.md
	  │   ├── 2019-05-29-hello-world.md
	  │   └── 2019-05-30-welcome.md
	  ├── docs                             // 文档目录
	  │   ├── doc1.md
	  │   ├── doc1.md
	  │   ├── doc3.md
	  │   └── mdx.md                       // 支持 mdx 哦
	  ├── docusaurus.config.js             // 项目的全局配置信息、插件的配置
	  ├── node_modules
	  ├── package.json                     // node.js的工程配置文件（帮助你生成并部署网站的脚本）
	  ├── sidebars.js                      // 配置文档页面侧边栏，只有文档页面才有，用于自定义文档的目录结构
	  ├── src                              // 页面或自定义的 React 组件目录
	  │   ├── css                          // 存放全局自定义样式
	  │   └── pages                        // 存放页面信息和react组件。  
	  │       └── index.js                 // 首页文件
	  ├── static                           // 静态文件目录，图片、字体等资源
	  └── yarn.lock
	  
	  ```
- ## 四、博客编写
  collapsed:: true
	- 写博客就是在blog里边创建一个markdown文件，标题开始部分是一个日期格式，Docusaurus 会自动把这个日期解析成咱们这个博客的发表日期，后边跟着这个文件的名字例如文件名：
		- ```
		  2022-01-20-onepage.md
		  ```
	- 文件第一段配置
		- ```
		  id:welcome      // 访问这个博客的url
		  title:Welcome   // 标题
		  author:xu       // 作者
		  author_title:就是作者的简短自我介绍
		  author_image_url:头像
		  tags:博客标签是个数组形式
		  ```
		-
		-
- ## 五、[页面开发](https://docusaurus.io/zh-CN/docs/creating-pages)
	- 入口函数
		- ```
		  export default function 标记
		  ```
	- 创建Docusaurus页面
		-
- 参考文献
	- [使用 Docusaurus 搭建个人网站项目](https://blog.csdn.net/weixin_47872288/article/details/124887877)
-