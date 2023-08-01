title:: kt高阶-T.repeat（）函数

- # 作用
	- 首先，我们从这个函数名就可以看出是关于`重复`相关的一个函数，再看起源码，从源码的实现来说明这个函数的作用：
	- **[[#red]]==根据传入的重复次数去重复执行一个我们想要的动作(函数)*==*
- # 源码
	- ```kotlin
	  public inline fun repeat(times: Int, action: (Int) -> Unit) {
	      contract { callsInPlace(action) }
	  
	      for (index in 0..times - 1) {
	          action(index)
	      }
	  }
	  ```
- # 示例
	- ```kotlin
	  repeat(5){
	      println("我是重复的第${it + 1}次，我的索引为：$it")
	  }
	  
	  我是重复的第1次，我的索引为：0
	  我是重复的第2次，我的索引为：1
	  我是重复的第3次，我的索引为：2
	  我是重复的第4次，我的索引为：3
	  我是重复的第5次，我的索引为：4
	  ```