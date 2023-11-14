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
		- 作用：==导航行为是全局的==，可以在整个应用的不同部分都有可能调用，并且可能导航到多个不同的目的地
	- 2、<action放在<fragment
		-
		- 作用：Fragment 内触发导航