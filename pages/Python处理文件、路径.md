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
		  C:pythonProject
		  	|-MigrateToAndroidX
		      	|-migrate.py
		  
		  ```
	- 在pythonProject下面执行python  MigrateToAndroidX/migrate.py，这时migrate.py获取当前路径对应的值其实是：
	- os.getcwd()：当前脚本的执行目录
		- ```
		  /Users/xuwenbin/PycharmProjects/pythonProject
		  
		  ```
	- sys.path[0]或sys.argv[0]
		- ```
		  MigrateToAndroidX/migrate.py
		  ```
		- 取的是被初始执行的脚本的所在目录，非完整路径
	- os.path.split(os.path.realpath(__file__))[0]
		- ```
		  /Users/xuwenbin/PycharmProjects/pythonProject/MigrateToAndroidX
		  ```
		- 取的是file所在文件.py的所在目录
-