- ## 示例1：该方法使用了高阶函数，将函数返回值会返回
  collapsed:: true
	- ```kotlin 
	   fun click1(view: View) = runBlocking { // 外协程
	          // 协程环境了  默认运行在当前线程。如 main
	          launch(Dispatchers.IO) { // 内协程  ,进行线程切换，到io线程
	              Log.e("click1: ", "launch ${Thread.currentThread().name}")
	  			// 睡 10s
	              repeat(10) {
	                  Thread.sleep(1000)
	                  Log.e("click1: ", "计数中: ${it}")
	              }
	          }
	      }
	  ```
- ## 示例2：使用协程前后对比
	- ## 不使用的代码，开线程进行耗时，切主线程更新ui
	  collapsed:: true
		- ```kotlin
		  private val mOkHttpClient: OkHttpClient = OkHttpClient()
		  private val mRequest = Request.Builder().url("https://baidu.com").get().build()
		  
		  // TODO 同学们，这个是不使用协程的   是使用传统写法
		  fun displayMethod(textView: TextView) {
		  
		      // 主线程 更新UI
		      val han = Handler(Looper.getMainLooper()) {
		          textView.text = it.obj as String
		  
		          false
		      }
		  
		      // 异步线程
		      object: Thread() {
		          override fun run() {
		              super.run()
		  
		              // Thread.sleep(2000)
		  
		              // 不考虑异常的情况
		              val result = mOkHttpClient.newCall(mRequest).execute().body()?.string()
		              val msg =  han.obtainMessage()
		              msg.obj = result
		              han.sendMessage(msg)
		          }
		      }.start()
		  
		  
		  
		  }
		  ```
	- ## 使用协程写法
	  collapsed:: true
		- ```kotlin
		  fun displayMethodOk(textView: TextView) = runBlocking {
		      launch {
		          // 默认当前线程main
		  
		          // 默认还是 main
		          // Dispatchers.IO  异步  async 函数能返回job Deferred能拿到协程异步执行后的结果
		          val def = async(Dispatchers.IO) {
		  
		              // 异步线程
		  
		              true
		              "String"
		  
		              // 不考虑异常的情况
		              mOkHttpClient.newCall(mRequest).execute().body()?.string()
		          }
		  
		          // main
		          // 可以拿到异步执行后的结果
		          textView.text = def.await()
		      }
		  }
		  ```
		- [[协程-async函数]]
	- ## 简化版本写法
	  collapsed:: true
		- ```kotlin
		  fun displayMethodOk(textView: TextView) = runBlocking {
		  
		      // 下面是简化
		      launch {
		          textView.text = async(Dispatchers.IO) {
		              mOkHttpClient.newCall(mRequest).execute().body()?.string()  // 异步的
		          }.await()
		      }
		  }
		  ```
- ## 示例3：像rxjava一样 doOnNext（）实现线程的来回切换
	- ```kotlin
	  fun click3(view: View) = runBlocking {
	          // TODO 完成这种  异步线程  和  主线程  的切换，  这个需求：之前我们用RxJava实现过了哦
	          // 1.注册耗时操作
	          // 2.注册耗时操作完成后，更新注册UI
	          // 3.登录耗时操作
	          // 4.登录耗时操作完成后，更新登录UI
	  
	          // main
	          launch {
	  			// 这个runBlocking 为阻塞式的，整个协程体执行完，才会弹窗
	              val pro = ProgressDialog(this@MainActivity)
	              pro.setMessage("正在执行中...")
	              pro.show()
	              // withContext 在指定调度器上运行协程
	              withContext(Dispatchers.IO) {
	                  // 1.注册耗时操作  异步
	                  Log.d("click3", "1.注册耗时操作: ${Thread.currentThread().name}")
	                  Thread.sleep(2000)
	              }
	  
	              // 2.注册耗时操作完成后，更新注册UI  main
	              Log.d("click3", "2.注册耗时操作完成后，更新注册UI: ${Thread.currentThread().name}")
	  
	              withContext(Dispatchers.IO) {
	                  // 3.登录耗时操作  异步
	                  Log.d("click3", "3.登录耗时操作: ${Thread.currentThread().name}")
	                  Thread.sleep(3000)
	              }
	  
	              // 4.登录耗时操作完成后，更新登录UI
	              Log.d("click3", "4.登录耗时操作完成后，更新登录UI: ${Thread.currentThread().name}")
	  
	              // pro.dismiss()
	          }
	  
	      }
	  ```
	- ## 注意
		- 这个runBlocking 为阻塞式的，整个协程体执行完，才会弹窗。非阻塞式见GlobalScope
	- [[GlobalScope使用示例]]
-