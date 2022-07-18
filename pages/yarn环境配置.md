- 1、执行安装
	- ```
	  curl -o- -L https://yarnpkg.com/install.sh | bash
	  ```
- 2、报错的话添加提示中的环境变量
	- ```
	  > WARNING: GPG is not installed, integrity can not be verified!
	  > Extracting to ~/.yarn...
	  > Adding to $PATH...
	  > We've added the following to your /Users/xuwenbin/.zshrc
	  > If this isn't the profile of your current shell then please add the following to your correct profile:
	     
	  export PATH="$HOME/.yarn/bin:$HOME/.config/yarn/global/node_modules/.bin:$PATH"
	  
	  Yarn requires Node.js 4.0 or higher to be installed.
	  > Yarn was installed, but doesn't seem to be working :(.
	  ```
	- 执行
		- ```
		  export PATH="$HOME/.yarn/bin:$HOME/.config/yarn/global/node_modules/.bin:$PATH"
		  ```
- 3、提示需要安装Node.js：Yarn requires Node.js 4.0 or higher to be installed.
	- [Nodejs](https://nodejs.org/en/)
	-
-