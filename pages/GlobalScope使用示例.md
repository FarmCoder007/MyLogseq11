- ## 示例1、支持挂起，非阻塞式，
	- ```kotlin
	   fun click4(view: View)  {
	          // TODO 完成这种  异步线程  和  主线程  的切换，  这个需求：之前我们用RxJava实现过了哦
	          // 1.注册耗时操作
	          // 2.注册耗时操作完成后，更新注册UI
	          // 3.登录耗时操作
	          // 4.登录耗时操作完成后，更新登录UI
	  
	          // main
	          GlobalScope.launch (Dispatchers.Main) {
	  
	              val pro = ProgressDialog(this@MainActivity)
	              pro.setMessage("正在执行中...")
	              pro.show()
	  
	              withContext(Dispatchers.IO) {
	                  // 1.注册耗时操作  异步
	                  Log.d("click3", "1.注册耗时操作: ${Thread.currentThread().name}")
	                  Thread.sleep(2000)
	              }
	  
	              // 2.注册耗时操作完成后，更新注册UI  main
	              Log.d("click3", "2.注册耗时操作完成后，更新注册UI: ${Thread.currentThread().name}")
	              textView.text = "注册成功，你可以去登录了"
	              pro.setMessage(textView.text.toString())
	  
	              withContext(Dispatchers.IO) {
	                  // 3.登录耗时操作  异步
	                  Log.d("click3", "3.登录耗时操作: ${Thread.currentThread().name}")
	                  Thread.sleep(3000)
	              }
	  
	              // 4.登录耗时操作完成后，更新登录UI
	              Log.d("click3", "4.登录耗时操作完成后，更新登录UI: ${Thread.currentThread().name}")
	              textView.text = "登录成功，欢迎回来"
	              pro.setMessage(textView.text.toString())
	  
	              pro.dismiss()
	          }
	  
	      }
	  ```
- ## 示例2、[[挂起后的流程]]
	-
- ## 示例3、GlobalScope相当于守护线程
	- main函数 或者 application结束 它也结束
	- ```kotlin
	  fun main() {
	      // 非阻塞， 类似与 守护线程
	      GlobalScope.launch {
	          delay(1000)
	          println("1111111111111")
	      }
	  
	      println("A")
	  
	      // main 线程 睡眠 2秒
	      
	  	 Thread.sleep(200)
	      println("B")
	  
	      // main 结束
	  }
	  
	  执行结果
	  A
	  B
	  ```
	- 执行
		- 1、走协程里delay(1s),则会挂起、
		- 2、打印A
		- 3、主线程睡眠200ms
		- 4、打印B main函数执行完，程序退出了 协程也没了 11111不会打印
		-