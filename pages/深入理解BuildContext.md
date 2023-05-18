- ## 前言
	- 在flutter中，我们要开发一个UI界面，需要通过组合其它Widget 来实现，Flutter中，一切都是widget，当UI发生变化时，我们不去直接修改DOM，通过更新状态，
	- 让Flutter UI框架来根据新的状态来构建UI。
	- StatelessWidget（无状态的组件） 和 StatefulWidget（有状态的组件） 的build方法都会传一个BuildContext 对象
	  collapsed:: true
		- ```
		  @override
		    Widget build(BuildContext context) {
		  }
		  ```
	- 大多数时候，我们也需要使用context 做一些事，比如：
	  collapsed:: true
		- ```
		  Theme.of(context) //获取主题
		  Navigator.push(context, route) //入栈新路由
		  Navigator.pop(context); //导航到新页面，或者返回到上个页面
		  Scaffold.of(context)//提示信息组件
		  ```
	- BuildContext乱用引发的问题
	  collapsed:: true
		- ```
		  Scaffold.of() called with a context that does not contain a Scaffold.
		  
		  No Scaffold ancestor could be found starting from the context that was passed to Scaffold.of(). This usually happens when the context provided is from the same StatefulWidget as that whose build function actually creates the Scaffold widget being sought.
		  ```