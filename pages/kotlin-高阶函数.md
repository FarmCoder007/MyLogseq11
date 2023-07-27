- ## 1、初级高阶函数玩法，作为参数传入方法，可以用来回调，代替接口回调
	- ```kotlin
	  fun main() {
	      show(true) {
	          // println(it)
	          'M'
	      }
	  }
	  
	  // TODO  loginMethod:(Boolean)->Unit
	  
	  //fun loginMethod(b: Boolean) : Unit {
	  //
	  //}
	  
	  // loginMethod 方法名
	  // (Boolean)方法的那个括号，方法入参为boolean类型
	  // -> 方法体 干活
	  // Char 为返回值类型
	  
	  fun show(isLogin: Boolean, loginMethod:(Boolean)->Char) {
	      if (isLogin) {
	          println("登录成功")
	          val r = loginMethod(true)
	          println(r)
	      } else {
	          println("登录失败")
	          loginMethod(false)
	      }
	  }
	  ```