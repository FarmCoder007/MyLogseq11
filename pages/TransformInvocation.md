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
  collapsed:: true
	- 1、isIncremental()：
		- 当前 Transform 任务是否增量构建；
	- 2、getInputs()：
		- 获取 TransformInput 对象，它是消费型输入内容，对应于 Transform#getScopes() 定义的范围；
	- 3、getReferencedInputs()：
		- 获取 TransformInput 对象，它是引用型输入内容，对应于 Transform#getReferenceScope() 定义的内容范围；
	- 4、getOutPutProvider()：
		- TransformOutputProvider 是对输出文件的抽象。
	-