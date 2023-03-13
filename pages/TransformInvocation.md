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
	-