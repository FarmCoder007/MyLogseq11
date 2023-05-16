- # 一、背景
	- MetaX组件化中有一个很重要的特点，能够在编译期知道RD修改了哪些api方法，这样在修改或删除方法时会给出相应的提示并中断编译。通过在整个开发期间不修改api方法，能够保证aar接口向下兼容，从而达到了QA回归仅限新增api的模块，有效提升验证回归效率。
	- 本文中API变化是指aar库的接口或类向外暴露public 方法发生了名称、返回值、参数等变化，并不会检查方法内部实现的变化。
	- 如何在编译时简单准确的获取当前变更内容，是我们亟待解决的问题。
- # 二、实现
	- ## 定义数据
		- 提到API变化，一定是有一个固定参考对象做对比才能得出变化结果。因此我们将首次使用MetaX运行时生成的类大纲集合文件是否存在做为参考点。
		- 类大纲文件记录了当前类或接口的public方法信息，以及当前文件的相对路径、语言方式是java或kotlin、class还是接口枚举之类的
		- ```json
		  //kotlin 单例object的类大纲信息
		  {
		    "path": "demo-api/src/main/java/com/metax/demo/api/utils/DeviceInfoUtils.kt",
		    "package": "com.metax.demo.api.utils",
		    "methods": [
		      {
		        "p": "Nullable(Context)",
		        "r": "Nullable(String)",
		        "name": "fun getImei(Nullable(Context)):Nullable(String)",
		        "n": "getImei"
		      }
		    ],
		    "name": "DeviceInfoUtils",
		    "language": "kotlin",
		    "type": "object"
		  }
		  ```
		- 如果本地未获取到对应的类大纲集合文件，则认为是首次生成，然后会遍历当前工程的sourceSet目录，生成工程的类大纲集合文件。在MetaX框架中这个文件被命名为[[#red]]==api-public-method.json==，放在工程的根目录中，会随着Git提交到[[#red]]==master主分支==。另外在RD每次合并当前需求分支时，会伴随的api变更检测Task自动重新生成当前类大纲集合文件，重新覆盖旧文件。这样就能解决数据源参考唯一性的问题。
		- 所以每次是需求分支的类大纲和master分支的类大纲做对比，得出api
-