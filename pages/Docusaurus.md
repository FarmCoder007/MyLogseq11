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
	- 5-1、入口函数
		- ```
		  export default function 标记
		  ```
	- 5-2、创建Docusaurus页面
		- 1、创建文件 /src/pages/helloReact.js：
		  collapsed:: true
			- ```
			  import React from 'react';
			  import Layout from '@theme/Layout';
			  
			  export default function Hello() {
			    return (
			      <Layout title="Hello" description="Hello React Page">
			        <div
			          style={{
			            display: 'flex',
			            justifyContent: 'center',
			            alignItems: 'center',
			            height: '50vh',
			            fontSize: '20px',
			          }}>
			          <p>
			            修改 <code>pages/helloReact.js</code>，然后保存，页面会重载。
			          </p>
			        </div>
			      </Layout>
			    );
			  }
			  ```
		- 2、保存后，开发服务器会自动刷新，呈现更改。 现在打开 http://localhost:3000/helloReact  你就能看到刚刚创建的新页面了
	- 5-3、添加 Markdown 页面
		- 1、创建文件 /src/pages/helloMarkdown.md：
		  collapsed:: true
			- ```
			  ---
			  title: 我的问候页面标题
			  description: 我的问候页面描述
			  hide_table_of_contents: true
			  ---
			  
			  # 你好
			  
			  今天过得怎么样？
			  ```
		- 2、同样地，新页面会被创建在 http://localhost:3000/helloMarkdown 处。Markdown 不如 React 页面灵活，因为它总是会用主题布局。
	-
- 参考文献
	- [使用 Docusaurus 搭建个人网站项目](https://blog.csdn.net/weixin_47872288/article/details/124887877)
-