- 对于http来说  核心是 method  path  body，而header 是对这些数据进行辅助的【比如 数据有多长  有没有压缩】
- # 1、[[#red]]==**host 是目标主机地址**==
	- DNS 查询： 域名系统   根据域名 查找ip地址的
	- host ⽬标主机 : 它不是在⽹络上查找ip 寻址的，⽽是在⽬标服务器上⽤于定位⼦服务 器的
		- 比如 在阿里云申请一个主机，在这个主机上运行了4个服务器     man.com      woman.com      son.com      brother.com
- # 2、[[#red]]==**Content-Type  指定  Body  的类型**==
	- ## 1、[[#red]]==**text/html：html文本**==
		- 用于浏览器页面的响应【比如请求个html,返回的响应报文里的header 这个Content-Type 就是 text/html 】
	- ## 2、[[#red]]==**application/x-www-from-urlencoded: 普通表单类型**==
	  collapsed:: true
		- 【纯文字的表单 ，键值对的方式，配置到body里 】  是 encoded Url 格式    [[#red]]==**搭配post使用**==
		- 格式：
			- ```java
			  POST /users HTTP/1.1
			  Host: api.github.com
			  Content-Type: application/x-www-form-urlencoded
			  Content-Length: 27
			  // 表单内容
			  name=xiaoxiao&gender=male
			  ```
		- ## Retrofit中的使用
			- @post   @FormUrlEncoded  搭配 @Field使用的
			- 上边生成表单的键值对 的键   用@Field标记
				- ```java
				     @FormUrlEncoded
				     @POST("/users")
				     Call<User> addUser(@Field("name") String name, @Field("gender") String gender);
				  ```
	- ## 3、multipart/form-data: 多部分形式的表单，
	  collapsed:: true
		- 一般用于传输包含二进制内容的多项内容     【如传文件  传图片 加文字】
		- 【传图片的标准方法之一】
			- ![](https://img-blog.csdnimg.cn/20210413163657329.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1d2IxMjN4dXdi,size_16,color_FFFFFF,t_70)
			- 生成请求报文时 content-Type   为如下  多个 boundary  作用：区分不同类型分块的   在文字 和图片 之间会有个这个分隔符
			  collapsed:: true
				- ```java
				  POST /users HTTP/1.1
				  Host: hencoder.com
				  Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
				  Content-Length: 2382
				   
				   
				   
				  ------WebKitFormBoundary7MA4YWxkTrZu0gW
				  Content-Disposition: form-data; name="name"
				   
				   
				  ------WebKitFormBoundary7MA4YWxkTrZu0gW
				  Content-Disposition: form-data; name="avatar";
				  filename="avatar.jpg"
				  Content-Type: image/jpeg
				   
				   
				  JFIFHHvOwX9jximQrWa......
				  ------WebKitFormBoundary7MA4YWxkTrZu0gW--
				  ```
			- 对应Retrofit的实现方式
				- ```java
				  @Multipart
				  @POST("/users")
				  Call<User> addUser(@Part("name") RequestBody name,@Part("avatar") RequestBody avatar);
				   
				  ...
				  RequestBody namePart =RequestBody.create(MediaType.parse("text/plain"),nameStr);
				  RequestBody avatarPart =RequestBody.create(MediaType.parse("image/jpeg"),avatarFile);
				  api.addUser(namePart, avatarPart);
				  ```
	- ## 4、[[#red]]==**application/json： json 形式**==
	  collapsed:: true
		- 用于web api的响应         或者  post/ put请求
		- ## 对应 retrofit代码：
		  collapsed:: true
			- ```java
			  @POST("/users")
			  Call<User> addUser(@Body("user") User user);
			  ...
			  // 需要使⽤ JSON 相关的 Converter
			  api.addUser(user);
			  ```
	- ## 5、[[#red]]==**image/jpeg     application/zip  ：  传输单文件**==
	  collapsed:: true
		- 用于web api 响应 【从服务器下载东西 响应里的header 里 Content-Type 就是这个 】 或者   post/put的请求 【上传图片  标准方法二】
		  collapsed:: true
			- ```java
			  POST /user/1/avatar HTTP/1.1
			  Host: hencoder.com
			  Content-Type: image/jpeg
			  Content-Length: 1575
			  JFIFHH9......
			  ```
		- 对应的retrofi  实现：
			- ```java
			  @POST("users/{id}/avatar")
			  Call<User> updateAvatar(@Path("id") String id, @BodyRequestBody avatar);
			  ...
			  RequestBody avatarBody =RequestBody.create(MediaType.parse("image/jpeg"),avatarFile);
			  api.updateAvatar(id, avatarBody
			  ```
- # 3、==**Content-Length:你发送的body的长度**==。
	- 让服务器知道接收多长 算结束
	- ** 作用： 如果 使用  结束符号作为结束        有可能在 body里数据 和 结束符合冲突了    如  结束符定位/     但是 我想传带/ 的数据那么这个数据就会被截断了 **
- # 4、与登录授权有关的header
	- 1、cookie/Set-Cookie : 发送和 设置cookie
	- 2、Authorization： 授权信息
- # >[[#red]]==**以下了解**==
- # 5、Location    指定重定向的⽬标 URL
- # 6、User-Agent⽤户代理，即是谁实际发送请求、接受响应的，例如⼿机浏览器、某款⼿机 App。
- # 7、Accept: 客户端能接受的数据类型。如 text/html
- # 8、Accept-Charset: 客户端接受的字符集。如 utf-8
- # 9、Accept-Encoding: 客户端接受的压缩编码类型。如 gzip
- # 10、Content-Encoding ：压缩类型。如 gzip
-