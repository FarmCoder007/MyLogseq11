- os.system() 是 Python 内置的一个函数，用于在操作系统上执行命令。它的语法如下
	- ```
	  os.system(command)
	  ```
- 其中，command 是要执行的命令，可以是一个字符串或一个包含命令和参数的列表。执行命令时，os.system() 函数会阻塞程序的执行，直到命令执行完成，并返回命令的退出状态码。
- ## 例如：拉取文件Python os.system("curl " + url + " >> xxx.html")
	- 这行 Python 代码使用了 os.system() 函数和 Shell 命令 curl 来将某个 URL 的内容追加到文件 xxx.html 中。
	- ```python
	  url = "https://sdl.58cdn.com.cn/productor/2016/58clientBack/58clientBack/MetaX/androidx-source.html"
	  if __name__ == "__main__":
	  	 # 加载映射资源,,把url下载到source文件上
	      os.system("curl " + url + " >> source.html")
	  
	  ```