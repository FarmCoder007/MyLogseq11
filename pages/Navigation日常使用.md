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
	- 1、