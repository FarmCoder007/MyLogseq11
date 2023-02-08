- 一、类的声明 class关键字
	- ```
	  
	  class ReportModule:
	      # 构造方法
	      def __init__(self):
	          self.code = []
	          self.res = []
	          self.gradle = []
	          self.rules = []
	  ```
- 二、类的实例化
	- ```
	  ReportModule()
	  ```
- 三、错误使用
	- ```
	  # 这样声明类。变量为集合
	  class ReportModule:
	          code = []
	          res = []
	          gradle = []
	          rules = []
	  # 创建2个实例 。实例不同，但是类是同一个 变量code，res等列表是同一个，故修改a的code列表 会影响b
	  a = ReportModule()
	  b = ReportModule()
	  
	  参考文档：
	          
	  ```
- 参考
	- [Python类中的List在两个不同的实例上共享同一个对象？](https://www.cnpython.com/qa/87166)