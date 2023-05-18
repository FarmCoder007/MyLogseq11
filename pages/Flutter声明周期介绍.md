- ## 背景
  collapsed:: true
	- 作为 Android 开发者，我们对 Activity、Service 等系统组件的生命周期非常熟悉，在日常开发中也会利用组件的生命周期特点开发业务逻辑。
	- Flutter 中有“一切皆Widget”的说法，Widget 是 Flutter 功能的抽象描述，是视图的配置信息，也是数据的映射。Android 开发中的 View、Layout、Activity、Application等，对应到 Flutter 中都是 Widget。
	- Flutter 的 Widget 有 StatelessWidget 和 StatefulWidget 两种类型，StatelessWidget 用于处理静态的、无状态变化的视图展示，而 StatefulWidget 应对有交互、需要动态变化视觉效果的场景。
	- StatelessWidget 通过父 Widget 初始化时传入的静态配置就能完全控制其静态展示，而 StatefulWidget 需要借助于 State 对象，在特定阶段来处理用户交互或内部数据的变化，然后更新到 UI 上。这些特定的阶段涵盖了一个 Widget 从初始化到卸载的全过程，即生命周期。Flutter 中的生命周期即指的 StatefulWidget 的生命周期，并通过 State 对象来体现，StatelessWidget 的生命周期只有 build 过程，且只会执行一次。
- ## 生命周期方法
	- Flutter中的生命周期方法主要包括以下内容：
	- createState
	  collapsed:: true
		- 该函数为 StatefulWidget 中创建 State 的方法，当 StatefulWidget 被调用时会立即执行 createState 。
		- ```
		  class MyPage extends StatefulWidget {
		    @override
		    _MyPageState createState() => _MyScreenState();
		  }
		  ```
	- initState
	  collapsed:: true
		- 该函数为 State 初始化调用，因此可以在此期间执行 State 各变量的初始赋值，同时也可以在此期间与服务端交互，获取服务端数据后调用 setState 来设置 State。
		- ```
		  int a;
		  @override
		  void initState() {
		    a = 0;
		    super.initState();
		  }
		  ```
	- didChangeDependencies
	  collapsed:: true
		- 用来专门处理 State 对象依赖关系的变化，这里说的 State 为全局 State ，例如语言或者主题等，会在 initState() 方法调用结束后被调用。
		- ```
		  @override
		  void didChangeDependencies() {
		  super.didChangeDependencies();
		  }
		  ```
	- build
	  collapsed:: true
		- 主要是构建视图，返回需要渲染的 Widget ，由于 build 会被调用多次，因此在该函数中只能做返回 Widget 相关逻辑，避免因为执行多次导致状态异常。
		- ```
		  @override
		  Widget build(BuildContext context, MyButtonState state) {
		    return Container(color:Colors.red);
		  }
		  ```
	- reassemble
	  collapsed:: true
		- 主要是提供开发阶段使用，在 debug 模式下，每次热重载都会调用该函数，因此在 debug 阶段可以在此期间增加一些 debug 代码，来检查代码问题。
	- didUpdateWidget
		- 该函数主要是在组件重新构建，比如说热重载，父组件发生 build 的情况下，子组件该方法才会被调用，其次该方法调用之后一定会再调用本组件中的 build 方法。
		-