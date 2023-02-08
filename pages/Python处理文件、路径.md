- python获取当前目录
	- ```
	  os.getcwd()
	  os.path.abspath('.')
	  os.path.abspath(os.path.dirname(__file__))
	  ```
- 获取上一级目录
	- ```
	  os.path.abspath('..')
	  os.path.abspath(os.path.dirname(os.path.dirname(__file__)))
	  os.path.abspath(os.path.dirname(os.getcwd()))
	  os.path.abspath(os.path.join(os.getcwd(), ".."))
	  ```
- 获取上上一级目录
	- ```
	  os.path.abspath('../..')
	  os.path.abspath(os.path.join(os.getcwd(), "../..")))
	  ```
- python获取运行脚本所在的目录
	- 目录层级：
		- ```
		  C:test
		  |-getpath
		      |-path.py
		      |-sub
		          |-sub_path.py
		  ```
	- C:\test下面执行python getpath/path.py，这时sub_path.py里面与各种用法对应的值其实是：
	- os.getcwd()
		- “C:\test”，取的是起始执行目录，当前脚本的执行目录
	-