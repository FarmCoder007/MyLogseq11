title:: Rxjava2.0线程切换

- # [[SubScribeOn给上面代码分配线程]]
- # [[ObserveOn()给下边代码分配线程]]
- 线程切换的特点,多次切换，哪个生效
	- ```java
	                // subsceOn(1)  // 会显示是这个线程的原因，上一层卡片是被1线程执行
	                  // subsceOn(2)
	                  // subsceOn(3)
	  
	                  // observeOn(A)
	                  // observeOn(B)
	                  // observeOn(C) // 终点是被C执行
	  ```