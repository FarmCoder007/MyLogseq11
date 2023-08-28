- 在一个项目中会包括着多个Activity，系统中使用任务栈来存储创建的Activity实例，任务栈是一种“后进先出”的栈结构。举个栗子，若我们多次启动同一个Activity。系统会创建多个实例依次放入任务栈中。当按back键返回时，每按一次，一个Activity出栈，直到栈空为止。当栈中无不论什么Activity。系统就会回收此任务栈。
- # 启动模式
	- # 1、**Standard 标准模式**
	  collapsed:: true
		- **说明：** Android创建Activity时的默认模式，假设没有为Activity设置启动模式的话，默觉得标准模式。[[#red]]==**每次启动一个Activity都会又一次创建一个新的实例入栈，无论这个实例是否存在**==。
		- **生命周期：**如上所看到的，每次被创建的实例Activity 的生命周期符合典型情况，它的onCreate、onStart、onResume都会被调用。
		- **举例：**此时Activity 栈中以此有A、B、C三个Activity，此时C处于栈顶，启动模式为**Standard 模式**。
		- 若在C Activity中加入点击事件，须要跳转到还有一个同类型的C Activity。结果是还有一个C Activity进入栈中，成为栈顶。
			- ![栈1.png](../assets/栈1_1692968879315_0.png)
	- # 2、**SingleTop 栈顶复用模式**
	  collapsed:: true
		- ## 特点
			- 1、[[#red]]==**须要创建的Activity已经处于栈顶时，此时会直接复用栈顶的Activity。不会再创建新的Activity**==；
			- 2、==**若须要创建的Activity不处于栈顶，此时会又一次创建一个新的Activity入栈，同Standard模式一样。**==
		- **生命周期：**若情况一中栈顶的Activity被直接复用时，它的onCreate、onStart不会被系统调用，由于它并没有发生改变。可是一个新的方法 **onNewIntent**会被回调（Activity被正常创建时不会回调此方法）。
		- **举例：**此时Activity 栈中以此有A、B、C三个Activity，此时C处于栈顶，启动模式为**SingleTop 模式**。情况一：在C Activity中加入点击事件，须要跳转到还有一个同类型的C Activity。结果是直接复用栈顶的C Activity。
		- 情况二：在C Activity中加入点击事件，须要跳转到还有一个A Activity。结果是创建一个新的Activity入栈。成为栈顶。
		- ![栈2.png](../assets/栈2_1692969037746_0.png)
	- # 3、**SingleTask 栈内复用模式**
	  collapsed:: true
		- **说明：**若须要创建的Activity已经处于栈中时，此时不会创建新的Activity，而是将存在栈中的Activity上面的其他Activity所有销毁，使它成为栈顶。
		- **生命周期：**同SingleTop 模式中的情况一同样。仅仅会又一次回调Activity中的 **onNewIntent**方法
		- **举例：**此时Activity 栈中以此有A、B、C三个Activity。此时C处于栈顶，启动模式为**SingleTask 模式**。
		- 情况一：在C Activity中加入点击事件，须要跳转到还有一个同类型的C Activity。结果是直接用栈顶的C Activity。情况二：在C Activity中加入点击事件，须要跳转到还有一个A Activity。
		- 结果是将A Activity上面的B、C所有销毁，使A Activity成为栈顶。
		- ![20170303205023925.png](../assets/20170303205023925_1692969140130_0.png)
	- # 4、**SingleInstance 单实例模式**
		- **说明：** SingleInstance比較特殊，是全局单例模式，是一种加强的SingleTask模式。它除了具有它所有特性外，还加强了一点：具有此模式的Activity仅仅能单独位于一个任务栈中。
		- 这个经常使用于系统中的应用，比如Launch、锁屏键的应用等等，整个系统中仅仅有一个！所以在我们的应用中一般不会用到。了解就可以。
		- **举例：**比方 A Activity是该模式，启动A后。系统会为它创建一个单独的任务栈，由于栈内复用的特性。兴许的请求均不会创建新的Activity，除非这个独特的任务栈被系统销毁。
- # **启动模式的使用方式**
	- ## **1. 在 Manifest.xml中指定Activity启动模式**
	- 一种静态的指定方法，在Manifest.xml文件里声明Activity的同一时候指定它的启动模式，这样在代码中跳转时会依照指定的模式来创建Activity。样例例如以下：
	- ```
	  <activity android:name="..activity.MultiportActivity" android:launchMode="singleTask"/>
	  ```
	- ## **2. 启动Activity时。在Intent中指定启动模式去创建Activity**
	- 一种动态的启动模式，在new 一个Intent后，通过Intent的addFlags方法去动态指定一个启动模式。样例例如以下：
	- ```
	  Intent intent = new Intent();
	        intent.setClass(context, MainActivity.class);
	        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
	        context.startActivity(intent);
	  ```
	- ---
	- **注意：**以上两种方式都能够为Activity指定启动模式，可是二者还是有差别的。
	- **（1）优先级**：动态指定方式即另外一种比第一种优先级要**高**，若两者同一时候存在，以另外一种方式为准。
	- **（2）限定范围**：第一种方式无法为Activity直接指定 **FLAG_ACTIVITY_CLEAR_TOP** 标识，另外一种方式无法为Activity指定 **singleInstance** 模式。
- # 三. Activity 的 Flags**
  collapsed:: true
	- 标记位既能够设定Activity的启动模式，如同上面介绍的，在动态指定启动模式，比方 **FLAG_ACTIVITY_NEW_TASK** 和 **FLAG_ACTIVITY_SINGLE_TOP** 等。它还能够影响Activity 的运行状态 ，比方 **FLAG_ACTIVITY_CLEAN_TOP** 和 **FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS** 等。
	- 以下介绍几个基本的标记位，切勿死记，理解几个就可以，须要时再查官方文档。
	- ### **1. FLAG_ACTIVITY_NEW_TASK**
	- 作用是为Activity指定 “**SingleTask**”启动模式。跟在AndroidMainfest.xml指定效果同样。
	- ### **2. FLAG_ACTIVITY_SINGLE_TOP**
	- 作用是为Activity指定 “**SingleTop**”启动模式，跟在AndroidMainfest.xml指定效果同样。
	- ---
	- ### **3. FLAG_ACTIVITY_CLEAN_TOP**
	- 具有此标记位的Activity，启动时会将与该Activity在同一任务栈的其他Activity出栈。一般与SingleTask启动模式一起出现。它会完毕SingleTask的作用。但事实上SingleTask启动模式默认具有此标记位的作用
	- ---
	- ### **4.FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS**
	- 具有此标记位的Activity不会出如今历史Activity的列表中，使用场景：当某些情况下我们不希望用户通过历史列表回到Activity时，此标记位便体现了它的效果。它等同于在xml中指定Activity的属性：
	- ```
	  android:excludeFromRecents="true"
	  ```
- # **四. 启动模式的实际应用场景**
	- 这四种模式中的Standard模式是最普通的一种，没有什么特别注意。而SingleInstance模式是整个系统的单例模式，在我们的应用中一般不会应用到。所以，这里就具体解说 **SingleTop** 和 **SingleTask**模式的运用场景：
	- ## **1. SingleTask模式的运用场景**
		- 最常见的应用场景就是保持我们应用开启后仅仅有一个Activity的实例。最典型的样例就是应用中展示的主页[[#red]]==**（Home页）**==。
		- 假设用户在主页跳转到其他页面，运行多次操作后想返回到主页，假设不使用SingleTask模式，在点击返回的过程中会多次看到主页，这明显就是设计不合理了。
	- ## **2. SingleTop模式的运用场景**
		- 假设你在[[#red]]==**当前的Activity中又要启动同类型的Activity**==，此时建议将此类型Activity的启动模式指定为SingleTop，能够降低Activity的创建，节省内存！
	- ## 3、**注意：复用Activity时的生命周期回调**
		- 复用Activity启动，**Activity跳转时携带页面參数的问题**
		- 不走onCreate  走onNewIntent
-