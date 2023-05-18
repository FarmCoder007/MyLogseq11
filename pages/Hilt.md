- ## 依赖注入方式
  collapsed:: true
	- 下面我们以一个登录页面为例，登录页面LoginActivity内部变量引用如下图：
	  collapsed:: true
		- ![image.png](../assets/image_1684407429904_0.png)
	- LoginActivity中需要引用LoginViewModel，LoginViewModel内部包含了UserRepository,Repository呢需要传入2中数据来源：UserLocalDataSource和UserRemoteDataSource
	  collapsed:: true
		- ![image.png](../assets/image_1684407443491_0.png)
	- LoginActivity 是登录流程的入口点，用户与 Activity 进行交互。因此，LoginActivity 需要创建 LoginViewModel 及其所有依赖项。
	- 以Repository看一下Repository类的2种设置DataSource方式：
	  collapsed:: true
		- ```
		  class UserRepository{
		    //方式1：内部直接初始化DataSource
		    val userLocalDataSource = UserLocalDataSource()
		    val userRemoteDataSource = UserRemoteDataSource()
		    //...todo...
		    fun getUserDataFromLocal(userID:String){
		      var user = userLocalDataSource.loadUserById(userID)
		      //...
		    }
		  }
		  ```
		- ```
		  //方式2：通过构造函数初始化DataSource
		  class UserRepository(val userLocalDataSource:UserLocalDataSource,
		                       val userRemoteDataSource:UserRemoteDataSource){
		    //...todo...
		    fun getUserDataFromLocal(userID:String){
		      var user = userLocalDataSource.loadUserById(userID)
		      //...
		    }
		  }
		  ```
	- 上面的2种方式最大的区别就是DataSource是由Repository类内部初始化实现还是由外部构造好后传入Repository。像这样通过构造函数的方式传入类对象，是依赖注入的方式。同样的使用setXXX()方法也是注入。
	- 另外我们熟悉的创建类型的设计模式如：1.Builder模式 2.工厂模式也是依赖注入。
	  collapsed:: true
		- ```
		  //bulider模式
		  val user = User.Builder()
		  	.name("Tom")
		  	.age(19)
		  	.build()
		  
		  //工厂模式
		  class UserFactory{
		    fun newUser:User{
		      val user = User()
		      user.name = "Tom"
		      user.age = 19
		      return user
		    }
		  }
		  ```
	- 这些都属于由外部来提供依赖的初始化，都是依赖注入，只是我们的说法不一样而已。
- ## 依赖注入优点
  collapsed:: true
	- 从外部初始化类对象这个角度来说，我们一直在使用依赖注入。那这样注入有什么好处呢。
	- ### 重用类以及分离依赖项
	  collapsed:: true
		- 首先从代码分层角度来说，使用注入类提高了类的重用性并且能够分离依赖项。更容易换掉依赖项的实现。由于控制反转，代码重用得以改进，并且类不再控制其依赖项的创建方式，而是支持任何配置。
	- ### 重构容易
		- 依赖项成为 API Surface 的可验证部分，因此可以在创建对象时或编译时进行检查，而不是作为实现详情隐藏。
-