- # 一、概念：延时初始化
	- ```
	  protected lateinit var mTitle: String
	  ```
	- 上面mTitle这个变量表示不为null，但是需要延迟初始化，如果在使用这个变量时没有初始化则会抛出异常；
- # 二、使用场景 比如延迟声明 findViewById的时候
- # 三、判断延迟初始化的变量是否初始化
	- ## 错误用法
		- 使用if(mTitle != null)来判断，kotlin 会弹出下面的提示 Condition ‘mTitle != null’ is always ‘true’
	- ## 正确用法
		- ```
		  if (!this::mTitle.isInitialized) {//没有初始化过
		              mTitle = "默认值"
		  }
		  ```
		- 需要注意的是变量前面一定要加上**"this::"**,不然会提示找不到"isInitialized"这个方法