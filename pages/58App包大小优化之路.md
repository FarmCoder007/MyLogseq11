- # 背景
	- APP包大小是应用优化的重要一项，更小的包体积意味着更高的下载转化率，更快的安装速度和更小的运行时内存。58同城APP对于包大小治理时间很长，经历了资源处理，代码优化，升级编译工具等一系列操作，同时又自研了包大小监控平台，对每次版本内包大小增长做到AAR级别监控。
	- 下面就主要介绍一下58同城APP在包大小缩减和监控方面是如果做的。
- # 包APK组成
  collapsed:: true
	- Android应用的apk文件其实是一个zip压缩包，下面以58APP 某个版本的安装包为例，将apk文件直接拖入到Android Studio中，此时Studio会分析apk内各文件大小，并依据每个文件及目录大小进行排序。可以看到apk主要是由res和assets为代表的资源文件，以及代码编译后的dex文件和lib库内的so文件为包大小组成的大头。
	  collapsed:: true
		- ![image.png](../assets/image_1684422746514_0.png)
	- 下面按包体积的占比介绍一下各模块主要是做什么的
	- res目录：res是resource的缩写，这个目录存放项目中的资源文件，比如占比很大的drawable文件夹以及常用的layout、values文件夹
	- lib目录:存放应用程序依赖的native库文件, .so的形式存在
	- assets目录：用于存放需要打包到APK中的静态文件。和res的不同点在于，assets目录支持任意深度的子目录，用户可以根据自己的需求任意部署文件夹架构，而且res目录下的文件会在.R文件中生成对应的资源ID，assets不会自动生成对应的ID(在不动业务逻辑，或者代码逻辑的情况下,针对此项很难优化)
	- classes.dex文件：dex文件是工程中java、kotlin源代码编译后生成的class文件经过dexBuilder工具生成的特殊可执行文件
	- resources.arsc文件：编译后的二进制资源文件，生成一个资源映射表
	- META-INF目录：保存应用签名信息，此处可验证APK的完整性，签名等
	- AndroidManifest.xml：应件文件配置信息
- # 优化项
	- # 代码优化
		- ## ProGuard
		  collapsed:: true
			- 在未经混淆的class文件中可以很直观的看到源代码信息，如类名、方法名、变量名等，这些 符号带有许多语义信息，很容易被反编译成 Java 源代码。为了防止出现这种情况，我们可以使用ProGuard对 Java 字节码进行混淆。
			- 代码混淆也被称为 花指令，它 将计算机程序的代码转换成一种功能上等价，但是难以阅读和直接理解的形式。混淆就是对发布出去的程序进行重新组织和处理，使得处理后的代码与处理前代码完成相同的功能，而混淆后的代码很难被反编译，即使反编译成功也很难得出程序的真正语义。ProGuard的 作用 不仅仅是 保护代码，它也有 精简编译后程序大小 的作用，其 通过缩短变量和函数名以及丢失部分无用信息等方式，能使得应用包体积减小。
			- ## ProGuard的作用有3点：
			  collapsed:: true
				- ## 压缩（Shrinking）
				  collapsed:: true
					- 默认开启，以减小应用体积，移除未被使用的类和成员，并且 会在优化动作执行之后再次执行，因为优化后可能会再次暴露一些未被使用的类和成员。我们可以使用如下规则来关闭压缩：
					- -dontshrink 关闭压缩
				- ## 优化（Optimization）
				  collapsed:: true
					- 默认开启，在 字节码级别执行优化，让应用 运行的更快。使用如下规则可进行优化相关操作：
					- ```
					  -dontoptimize 关闭优化
					  -optimizationpasses n 表示proguard对代码进行迭代优化的次数，Android一般为5
					  ```
				- ## 混淆（Obfuscation）
				  collapsed:: true
					- 默认开启，增大反编译难度，类和类成员会被随机命名，除非用 优化字节码 等规则进行保护。使用如下规则可以关闭混淆：
					- ```
					  -dontobfuscate 关闭混淆
					  ```
			- ## gradle配置
			  collapsed:: true
				- 混淆之后，默认会在工程目录 app/build/outputs/mapping/release 下生成一个 mapping.txt 文件，这就是 混淆规则，所以我们可以根据这个文件把混淆后的代码反推回原本的代码。要使用混淆，我们只需配置如下代码即可：
				- ```
				  buildTypes {
				      release {
				          // 1、是否进行混淆
				          minifyEnabled true
				          // 2、开启zipAlign可以让安装包中的资源按4字节对齐，这样可以减少应用在运行时的内存消耗
				          zipAlignEnabled true
				          // 3、移除无用的resource文件：当ProGuard 把部分无用代码移除的时候，
				          // 这些代码所引用的资源也会被标记为无用资源，然后
				          // 系统通过资源压缩功能将它们移除。
				          // 需要注意的是目前资源压缩器目前不会移除values/文件夹中
				          // 定义的资源（例如字符串、尺寸、样式和颜色）
				          // 开启后，Android构建工具会通过ResourceUsageAnalyzer来检查
				          // 哪些资源是无用的，当检查到无用的资源时会把该资源替换
				          // 成预定义的版本。主要是针对.png、.9.png、.xml提供了
				          // TINY_PNG、TINY_9PNG、TINY_XML这3个byte数组的预定义版本。
				          // 资源压缩工具默认是采用安全压缩模式来运行，可以通过开启严格压缩模式来达到更好的瘦身效果。
				          shrinkResources true
				          // 4、混淆文件的位置，其中 proguard-android.txt 为sdk默认的混淆配置，
				          // 它的位置位于android-sdk/tools/proguard/proguard-android.txt，
				          // 此外，proguard-android-optimize.txt 也为sdk默认的混淆配置，
				          // 但是它默认打开了优化开关。并且，我们可在配置混淆文件将android.util.Log置为无效代码，
				          // 以去除apk中打印日志的代码。而 proguard-rules.pro 是该模块下的混淆配置。
				          proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
				          signingConfig signingConfigs.release
				      }
				  }
				  ```
				- 58同城APP目前仅开启了混淆操作，对于zipAlign和shrinkResources并未开启，因为开启后会出现某些页面打开失败的情况。
				- 开启混淆是一个常规操作，这个举动能给我们应用带来极大的包大小收益。
		- ## AndResGuard
		  collapsed:: true
			- Android在构建过程中会根据资源生成R文件，里面包含了资源索引，使用该索引可以在最终生成的resources.arsc资源映射表中找到对应资源，对于开发者来说在代码中引用资源很方便。
			- 资源混淆主要为了混淆资源ID长度(例如将res/drawable/welcome.png混淆为r/s/a.png)，同时利用7z深度压缩，大大减少了安装包体积，同时提升了反破解难度。
			- 介绍微信的AndResGuard前，先看一下resource.arsc文件的结构，有助于我们更好的理解ResGuard原理
			  collapsed:: true
				- ![image.png](../assets/image_1684422937375_0.png)
			- resources.arsc一共有五种chunk类型：
			- –table，是整个reousces table的开始，它的chunksize即是整个文件的大小。
			- –package，指的是一个package的开始，其实在resources,arsc是可以有多个package的。而packageID即是资源resID的最高八位，一般来说系统android的是1(0x01)，普通的例如com.tencent.mm会是127(0x7f)，剩下的是从2开始起步。当然这个我们在aapt也是可以指定的(1-127即八位的合法空间,一些混合编译就是改这个packageID)。
			- –string, 代表stringblock，我们一共有三种类型的stringblock。分别是table stringblock,typename stringblock, specsname stringblock。
			- –type，这里讲的是typename stringblock里面我们用到的各种type(用到多少种类型的type,就有多少个type chunk)，例如attr, drawable, layout, id, color, anim等，Type ID是紧跟着Package ID。
			- –config, 即是Android用来描述资源维度，例如横竖屏，屏幕密度，语言等。对于每一种type，它定义了多少种config，它后面就紧跟着多少个config chunk,例如我们定义了drawable-mdpi,drawable-hdpi,那后面就会有两个config。
			- –entry，尽管没有entry这个chunk,但是每个config里面都会有很多的entry，例如drawable-mdpi中有icon1.png,icon2.png两个drawable,那在mdpi这个config中就存在两个entry。
			- ## ResTable_header(资源索引表头)
			  collapsed:: true
				- ![image.png](../assets/image_1684422959648_0.png)
				- ResTable_header的chunk(02 00 0C 00 6895 01 00 01 000000)
				  02 00 位置0~ 1,共2字节，表示该chunk的类型，值为0x0002表示类型为RES_TABLE_TYPE.
				  0C 00 位置2 ~ 3 ,共2字节，表示该chunk类型的头长度，
				  值为0x000C表示该类型的头长度为12字节长度。
				  6895 01 00位置4~7，共4字节，表示该chunk的总长度
				  ，值为0x00019568表示该chunk的总长度(这里为整个文件的大小)为103784字节长度。
				  01 000000 被编译的资源包的个数，这里只有一个。
			- ## ResStringPool_header(全局字符串资源)
				- 紧跟着资源索引表头部的是资源项的全局字符串资源,这个字符串资源池包含了所有的在资源包里面所定义的资源项的全局字符串,包括android工程中部分资源文件名（如res/drawable-hdpi/ic_launcher.png，res/layout/activity_main.xml等）及res/values/strings.xml中的字符串值
				- 上图简化如下：
				  collapsed:: true
					- ![image.png](../assets/image_1684422996650_0.png)
				- AndResGuard流程：
				  collapsed:: true
					- 1. table stringblock	把文件指向路径改变，例如res/layout/test.xml,改为res/layout/a.xml
					  2. 资源的文件名	需要将资源的文件名改为对应1，即将test.xml重命名为a.xml
					  3. specsname stringblock	旧的specsname除了白名单部分全部废弃，替换成所有我们混淆方案中用到的字符。由于大家都重复使用[a-z0-9_],specsname的总数量会大大减少。
					  4. entry中指向的specsname 中的id	例如原本test.xml它指向specsname中的第十项，我们需要用混淆后的a项的位置改写。
					  5. table chunk的大小	修改table chunk的最后大小
					  ![image.png](../assets/image_1684423019248_0.png){:height 826, :width 640}
		- ## R资源文件内联
			- Android 在构建的过程中，会为每个模块（库、应用）生成一份资源索引，诸如：Rid.class*，*Rlayout.class 等等，这对于开发者来说，在代码里引用资源十分的方便。
			- 为什么会出现内联？
			  在Library工程中引用的R资源索引不是final的不是常量，所以我们在Library工程不能在switch - case 和Annotation中使用资源索引。由于引用的资源不是final的，所以Library的产物aar中包含的class中使用的资源索引还是会以包名存在。
			- ![image.png](../assets/image_1684423043172_0.png)
			- 在App工程中构建时会将依赖的AAR资源进行合并，根据合并的结果生成最终的R资源索引，这时的资源索引已经确定，所以全部是final的，java编译器在编译时会将final常量进行inline内联操作，也就是App工程中的java源码编译后的class中使用的R资源索引全部会替换为常量值。但resource.arsc文件中有关library的R资源类型还依然存在。
			- 我们采用了业界比较优秀的方案ByteX R文件瘦身。他是通过编译时使用Transform处理R文件的class，收集R资源相关的信息。流程如下：
			- 首先默认当前R文件class是可以被删除的。
			  如果是静态final int的field并且没有在白名单中则添加到shouldBeInlinedRFields集合中，否则添加到shouldSkipInlineRFields集合中，标记当前class不能被删除。
			  如果当前R文件所有field没有在白名单中则添加到shouldDiscardRClasses集合中，也就是该R文件class是可以被删除的。
			  如果扫描到Styleable class的方法时，方法用于初始化静态变量和静态代码块，并且不需要保留该类时，会使用AnalyzeStyleableClassVisitor处理，如果数组大小和计算的大小一致则加到shouldBeInlinedRFields集合，否则抛出异常。
			  真正开始Transform处理R文件，扫描filed判断进行删除
			  58app在使用ByteX将R资源inline后，包大小减少了4.6M，dex数量从16个减到了11个。