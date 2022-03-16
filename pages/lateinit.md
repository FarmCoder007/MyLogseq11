- 一、概念：延时初始化
  collapsed:: true
	- ```
	  protected lateinit var mTitle: String
	  ```
	- 上面mTitle这个变量表示不为null，但是需要延迟初始化，如果在使用这个变量时没有初始化则会抛出异常；
- 二、使用场景 比如延迟声明 findViewById的时候
- 三、