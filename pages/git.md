- # 一、常用命令
	- 1 空目录 初始化git    git init
	  2 初始化后可以配置用户名和邮箱  
	     git config --global user.name "name"
	     git config --global user.email "邮箱"
	  3 没有公钥私钥前 可以生成  有就可以直接用
	    根据邮箱生成 ssh -keygen -t rsa -C “zengmianhui” 
	  4 将本地仓库关联远程仓库
	  git remote add origin   url
	  5 克隆  git clone  url