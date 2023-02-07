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
-