- ## 原函数
	- ```kotlin
	  /**
	   * 挂起函数，这里由于获取信息是后面依赖于前面，
	   * 所以使用挂起函数来解决Callback
	   * */
	  suspend fun testCoroutine(){
	      val user = getUserInfo()
	      val friendList = getFriendList(user)
	      val feedList = getFeedList(user,friendList)
	      println(feedList)
	  }
	  ```
- 在原来的`testCoroutine`方法中，我们调用了3次挂起函数，记住我们挂起函数的本质是挂起什么来着？挂起的是后面执行的代码，所以在`CPS`后，状态机逻辑会被分为4个部分：
	- ```kotlin
	  when(continuation.label){
	      0 -> {
	          //检测异常
	          throwOnFailure(result)
	          //将label设置为1，准备进入下一个状态
	          continuation.label = 1
	          //执行getUserInfo
	          suspendReturn = getUserInfo(continuation)
	          //判断是否挂起
	          if (suspendReturn == sFlag){
	              return suspendReturn
	          }else{
	              result = suspendReturn
	          }
	      }
	  
	      1 -> {        
	          throwOnFailure(result)
	          // 获取 user 值        
	           user = result as String        
	          // 将协程结果存到 continuation 里        
	          continuation.mUser = user        
	          // 准备进入下一个状态        
	          continuation.label = 2        
	          // 执行 getFriendList        
	          suspendReturn = getFriendList(user, continuation)        
	          // 判断是否挂起       
	          if (suspendReturn == sFlag) {            
	              return suspendReturn
	          }  else {            
	              result = suspendReturn
	          }           
	      } 
	      
	      2 -> {
	          throwOnFailure(result)
	          user = continuation.mUser as String
	          //获取friendList的值
	          friendList = result as String
	          //将挂起函数结果保存到continuation中
	          continuation.mUser = user
	          continuation.mFriendList = friendList
	          //准备进入下一个阶段
	          continuation.label = 3
	          //执行获取feedList
	          suspendReturn = getFeedList(user,friendList,continuation) 
	          //判断是否挂起
	          if (suspendReturn == sFlag){
	              return suspendReturn
	          }else{
	              result = suspendReturn
	          }
	      }
	      
	      3 -> {
	          throwOnFailure(result)
	          user = continuation.mUser as String        
	          friendList = continuation.mFriendList as String        
	          feedList = continuation.result as String        
	          loop = false
	      }
	  }
	  ```
- 这部分代码理解起来有点困难，我们来一步一步分析走一遍：
- ## 1
	- 第一次调用`testCoroutine`时，会给`continuation`赋值，为`TestContinuation`类型，默认情况下`continuaiton`中的`label`为0，所以会进入`when`的0分支。
- ## 2
	- 在`when`的0分支中，会先检查异常，然后将`continuaiton`的`label`设置为1，然后执行`getUserInfo`方法。由于该方法是真挂起函数，所以返回值`suspendReturn`和`sFlag`一样，这时`testCoroutine`会直接被`return`，即完成运行。
- ## 3
	- 在`getUserInfo`方法中，会模拟网络请求，当获取到网络请求数据后，会调用`getUserInfo`的`continuation`参数的`invokeSuspend(user)`方法，注意该方法的`continuation`和前一次调用`testCoroutine`的`continuation`是同一个。
- ## 4
	- 根据`TestContinuation`的定义，这时该`continuation`实例中的`result`就是获取到的`user`信息，然后`label`为1，然后开始第二次调用`testCoroutine`方法，同时参数依旧是这个`continuation`。
- ## 5
	- 第二次调用`testCoroutine`时，`continuaiton`不会再被创建，这时方法的`result`变量会保存`user`，会进入`when`的1分支里面。
- ## 6
	- 在1分支里，方法的`user`变量为获取到的用户信息的String类型，然后将该结果保存到`continuation`的`mUser`变量中。这时，我们`getUserInfo`方法的结果值就保存到了唯一`continuation`中，接着`label`设置为2，调用`getFriendList`方法，同样的该方法是挂起函数，这时第二次调用的`testCoroutine`方法被`return`掉。
- ## 7
	- 在`getFriendList`方法中，一样的逻辑，当获取到好友列表后，会回调唯一`continuaiton`的`invokeSuspend(friendList)`方法，这时`result`为好友列表信息，同时开启第三次调用`testCoroutine`方法。
- ## 8
	- 第三次调用`testCoroutine`时，会进入2分支，在该分支中，会给唯一`continuation`的`mFriendList`赋值好友列表信息，然后`label`设置为3，调用`getFeedList`挂起函数，这时第三次`testCoroutine`被`return`。
- ## 9
	- 在`getFeedList`中，回调唯一`continuation`的`invokeSuspend(feedList)`方法，这时`result`保存的是是动态信息，同时开始调用第四次`testCoroutine`方法。
- ## 10
	- 在第四次调用`testCoroutine`方法中，会进入3分支，在这里我们可以获取用户信息(保存在`continuation`的`mUser`字段中)、好友列表信息(保存在`continuation`的`mFriendList`字段中)和动态信息(保存在`result`)字段中。
- 到这里，一个调用3个挂起函数的挂起函数就分析完了，不得不说，设计的非常巧妙。利用唯一的`continuation`来保存挂起函数执行的值，通过多次调用函数自己来分割开来挂起函数。