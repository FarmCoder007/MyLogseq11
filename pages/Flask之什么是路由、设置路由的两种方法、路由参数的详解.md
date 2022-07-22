- # [教程](https://blog.csdn.net/weixin_45950544/article/details/103850670?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-3-103850670-null-null.pc_agg_new_rank&utm_term=app.root_path%20flask&spm=1000.2123.3001.4430)
- # [API:send_from_directory](https://tedboy.github.io/flask/generated/flask.send_from_directory.html)
- # 安装flask：
	- ```js
	  sudo pip install flask
	  ```
	- 如果报错 可先装 [[开发工具]] pip
- # 一、简介
	- 客户端（例如Web浏览器）把请求发送给Web服务器，Web服务器再把请求发送给Flask程序实例。程序实例需要知道对每个URL请求运行哪些代码，所以保存了一个URL到python函数的映射关系。处理URL和函数之间关系的程序称为路由。
	  在Flask 程序中定义路由的最简便方式，是使用程序实例提供的app.route 修饰器，把修饰的函数注册为路由。
- # 二、装饰器：@app.route()
	- ## 内容样式
		- ```
		  @app.route('/user/<name>') # 动态名
		  @app.route('/post/<int:post_id>') # 登录用户的<整数形式ID>
		  @app.route('/post/<float:post_id>') # 小数形式的ID
		  @app.route('/user/<path:path>') # 传递URL的路径
		  @app.route(''/login', methods=['GET', 'POST']')
		  ```
-