- # 一、背景
	- MetaX组件化中有一个很重要的特点，能够在编译期知道RD修改了哪些api方法，这样在修改或删除方法时会给出相应的提示并中断编译。通过在整个开发期间不修改api方法，能够保证aar接口向下兼容，从而达到了QA回归仅限新增api的模块，有效提升验证回归效率。
	- 本文中API变化是指aar库的接口或类向外暴露public 方法发生了名称、返回值、参数等变化，并不会检查方法内部实现的变化。
	- 如何在编译时简单准确的获取当前变更内容，是我们亟待解决的问题。
- # 二、实现
	- ## 定义数据
	  collapsed:: true
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
		- [[#red]]==所以每次是需求分支的类大纲和master分支的类大纲做对比，得出api变更范围==
	- ## 如何生成类大纲
		- 确定好了类大纲的格式后，下一步就是如何简单快速的生成这些类大纲数据。调研有3种方式：1. Python读取代码文件逐行分析 2. hook编译的Transform使用ASM来收集类信息 3.使用AST（抽象语法树）用java解释java
		- 使用Python读取源码文件分析是第一个被排除的，这个方案工程量大，对java和kotlin语言需要有高深的理解和运用才能自如处理各种case。下面主要分析ASM和AST两种方式
		- ![image.png](../assets/image_1684238760351_0.png)
		- ##ASM读取
		  在编译期hook Transform完全没有必要，且不可行，因为MetaX工程在编译时就需要知道当前api变化了哪些类以及变化类的细节。查询ASM使用方法可以得出只用传入源码文件就可以，内部会调用javac命令来生成class，然后进行遍历各种方法和字段。
		- 一个简单的ASM使用包括ClassReader ClassWriter ClassVisitor，由于我们不用对class处理插桩，只需要定义个ClassVisitor就可以了，主要处理方法就是visitMethod，通过不断的回调来记录当前传入类的方法，过滤后生成一个类大纲信息。