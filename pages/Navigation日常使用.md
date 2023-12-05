### 1、扩展Fragment获取NavController
collapsed:: true
	- 1-1、Fragment.kt
		- ```kotlin
		  public fun Fragment.findNavController(): NavController =
		      NavHostFragment.findNavController(this)
		  ```
	- 1-2、fragment里直接调用findNavController() 获取NavController
		- ```kotlin
		  class Register : Fragment() {
		  
		      override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,
		                                savedInstanceState: Bundle?): View? {
		          // Inflate the layout for this fragment
		          val view = inflater.inflate(R.layout.fragment_register, container, false)
		  
		          view.findViewById<Button>(R.id.signup_btn).setOnClickListener {
		              findNavController().navigate(R.id.action_register_to_registered)
		          }
		          return view
		      }
		  }
		  ```
- ### 2、导航图<action 放在 <navigation 下 和 <action放在<fragment区别，及使用
  collapsed:: true
	- 1、`<action>` 放在 `<navigation>`
		- 示例
		  collapsed:: true
			- ```xml
			  <!-- 在 navigation 中定义的全局 action -->
			  <navigation>
			      <action
			          android:id="@+id/globalAction"
			          app:destination="@id/destinationFragment" />
			      <!-- 其他 fragment 和 navigation... -->
			  </navigation>
			  
			  ```
		- 作用：==**导航行为是全局的**==，可以在整个应用的不同部分都有可能调用，并且可能导航到多个不同的目的地
	- 2、<action放在<fragment
		- 示例
		  collapsed:: true
			- ```xml
			  <!-- 在 fragment 中定义的局部 action -->
			  <fragment>
			      <action
			          android:id="@+id/localAction"
			          app:destination="@id/anotherFragment" />
			      <!-- 其他 fragment 和 navigation... -->
			  </fragment>
			  
			  ```
		- 作用：==**特定Fragment 内触发导航**==,导航行为仅在特定的 Fragment 内有意义，仅在该 Fragment 内触发导航
- ### 3、导航图自动生成 xxxDirections,是safeargs插件生成的
	- ```
	  plugins {
	      id 'androidx.navigation.safeargs.kotlin'
	  }
	  ```
- ### 4、导航到登录页同时清空回退栈
	- ### 代码方式
		- API
		  collapsed:: true
			- ```kotlin
			          @JvmOverloads
			          public fun setPopUpTo(
			              @IdRes destinationId: Int,
			              inclusive: Boolean,
			              saveState: Boolean = false
			          ): Builder {
			              popUpToId = destinationId
			              popUpToRoute = null
			              popUpToInclusive = inclusive
			              popUpToSaveState = saveState
			              return this
			          }
			  ```
		- ```kotlin
		  navController.navigate(R.id.nav_third_login, null,
		                          NavOptions.Builder()
		                              .setPopUpTo(R.id.mobile_navigation, false) // 弹出到目标 Fragment 并包括它
		                              .build())
		  ```
	- ### xml
		- 如果你想在导航到某个目标 Fragment 后，弹出之前的导航栈元素，你可以使用 `popUpTo` 和 `popUpToInclusive` 属性。这会弹出到指定目标之前的所有节点。
		- 示例
		- ```xml
		  <action
		      android:id="@+id/action_to_destination_fragment"
		      app:destination="@id/destinationFragment"
		      app:popUpTo="@+id/startFragment"
		      app:popUpToInclusive="false"/>
		  
		  ```
		- `popUpTo` 属性指定了要弹出到的目标节点，即 `startFragment`。
		- `popUpToInclusive` 设置为 `false`，表示在弹出时不包括目标节点 `startFragment` 自身。