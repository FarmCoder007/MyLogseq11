- ## 1、初级高阶函数玩法，作为参数传入方法，可以用来回调，代替接口回调
  collapsed:: true
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
- ## 2、覆盖操作
  collapsed:: true
	- ```kotlin
	      // 覆盖操作
	      // 先定义了一个函数
	      var m14 = {number: Int -> println("我就是m14  我的值: $number")}
	      // 这个覆盖了里边的函数 但是可以保留入参
	      m14 = {println("覆盖  我的值: $it")}
	      m14(99)
	  
	  ```