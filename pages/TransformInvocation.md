- 一、常用方法
  collapsed:: true
	- ```kotlin
	  public interface TransformInvocation {
	  		
	      //...
	  
	      // 消费型输入内容
	      Collection<TransformInput> getInputs();
	  
	      // 引用型输入内容
	      Collection<TransformInput> getReferencedInputs();
	    
	      //...
	  
	      // 输出信息
	      TransformOutputProvider getOutputProvider();
	  
	      // 是否增量构建
	      boolean isIncremental();
	  }
	  
	  ```
- 二、api
	- 1、isIncremental()：
		- 当前 Transform 任务是否增量构建；
	- 2、getInputs()：
		- 获取 TransformInput 对象，它是消费型输入内容，对应于 Transform#getScopes() 定义的范围；
			- 输入内容 TransformInput 由两部分组成：
			- 1、DirectoryInput 集合： 以源码方式参与构建的输入文件，包括完整的源码目录结构及其中的源码文件；
			- 2、JarInput 集合： 以 jar 和 aar 依赖方式参与构建的输入文件，包含本地依赖和远程依赖。
	- 3、getReferencedInputs()：
		- 获取 TransformInput 对象，它是引用型输入内容，对应于 Transform#getReferenceScope() 定义的内容范围；
	- 4、getOutPutProvider()：
		- TransformOutputProvider 是对输出文件的抽象。
		- 输出内容 TransformOutputProvider 有两个主要功能：
	-