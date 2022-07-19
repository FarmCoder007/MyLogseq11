- ## 一、简介
	- [Docusaurus](https://www.docusaurus.cn/docs/installation)是一个静态站点生成器。它构建了一个具有快速客户端导航的单页应用程序，利用React的全部功能使您的站点具有交互性。它提供开箱即用的文档功能，但可用于创建任何类型的网站（个人网站、产品、博客、营销登录页面等）
- ## 二、配置详解
	- ### [[docusaurus.config.js]]
		-
-
- ## 三、项目目录
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
  ├── package.json                     // node.js的工程配置文件
  ├── sidebars.js                      // 配置文档页面侧边栏，只有文档页面才有，用于自定义文档的目录结构
  ├── src                              // 页面或自定义的 React 组件目录
  │   ├── css                          // 存放全局自定义样式
  │   └── pages                        // 存放页面信息和react组件。  
  │       └── index.js                 // 首页文件
  ├── static                           // 静态文件目录，图片、字体等资源
  └── yarn.lock
  
  ```
- ## 博客编写
	- 写博客就是在blog里边创建一个markdown文件，标题开始部分是一个日期格式，Docusaurus 会自动把这个日期解析成咱们这个博客的发表日期，后边跟着这个文件的名字
		- ```
		  ```