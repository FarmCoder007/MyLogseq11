- 这里听着有点奇怪，挂起函数不就是要挂起吗 ？其实不然，当挂起函数内没有调用其他挂起函数或者实现挂起函数时，它就不需要挂起。
- 比如下面代码
	- ```kotlin
	  /**
	   * 获取用户信息
	   * */
	  suspend fun getUserInfo(): String{
	      withContext(Dispatchers.IO){
	          delay(1000)
	      }
	      return "Coder"
	  }
	  
	  ```
- 在`getUserInfo`方法中，当执行到`withContext`时，该方法就会被挂起，这时就会返回`CoroutineSingletons.COROUTINE_SUSPENDED`，用来标志`getUserInfo`被挂起了。
- 但是比如下面代码：
	- ```kotlin
	  /**
	   * 函数内部没有挂起
	   * */
	  suspend fun noSuspendGetUserInfo(): String{
	      return "zyh"
	  }
	  ```
- 在这种情况下，该函数就和普通函数一样，不会被挂起，同时IDE会提醒你这个`suspend`关键字是多余的。但是，IDE遇到`suspend`关键字时，就会发生`CPS`转换，所以上面方法经过`CPS`转换后，返回类型是`no suspend`的`String`类型，这也是一种伪挂起。
- 所以这里我们也就明白了为什么`CPS`后的挂起函数返回值类型是`Any?`了。