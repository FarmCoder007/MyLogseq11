# 背景
	- APP包大小是应用优化的重要一项，更小的包体积意味着更高的下载转化率，更快的安装速度和更小的运行时内存。58同城APP对于包大小治理时间很长，经历了资源处理，代码优化，升级编译工具等一系列操作，同时又自研了包大小监控平台，对每次版本内包大小增长做到AAR级别监控。
	- 下面就主要介绍一下58同城APP在包大小缩减和监控方面是如果做的。
- # [[包APK组成]]
- # 优化项
	- # 代码优化
		- ## ProGuard
		  collapsed:: true
			- 在未经混淆的class文件中可以很直观的看到源代码信息，如类名、方法名、变量名等，这些 符号带有许多语义信息，很容易被反编译成 Java 源代码。为了防止出现这种情况，我们可以使用ProGuard对 Java 字节码进行混淆。
			- 代码混淆也被称为 花指令，它 将计算机程序的代码转换成一种功能上等价，但是难以阅读和直接理解的形式。混淆就是对发布出去的程序进行重新组织和处理，使得处理后的代码与处理前代码完成相同的功能，而混淆后的代码很难被反编译，即使反编译成功也很难得出程序的真正语义。ProGuard的 作用 不仅仅是 保护代码，它也有 精简编译后程序大小 的作用，其 通过缩短变量和函数名以及丢失部分无用信息等方式，能使得应用包体积减小。
			- ## ProGuard的作用有3点：
			  collapsed:: true
				- ## 压缩（Shrinking）
					- 默认开启，以减小应用体积，移除未被使用的类和成员，并且 会在优化动作执行之后再次执行，因为优化后可能会再次暴露一些未被使用的类和成员。我们可以使用如下规则来关闭压缩：
					- -dontshrink 关闭压缩
				- ## 优化（Optimization）
					- 默认开启，在 字节码级别执行优化，让应用 运行的更快。使用如下规则可进行优化相关操作：
					- ```
					  -dontoptimize 关闭优化
					  -optimizationpasses n 表示proguard对代码进行迭代优化的次数，Android一般为5
					  ```
				- ## 混淆（Obfuscation）
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
		- ## [[微信的AndResGuard对resource.arsc文件混淆，减少大小]]
		- ## R资源文件内联
			- Android 在构建的过程中，会为每个模块（库、应用）生成一份资源索引，诸如：Rid.class*，*Rlayout.class 等等，这对于开发者来说，在代码里引用资源十分的方便。
			- 为什么会出现内联？
			  在Library工程中引用的R资源索引不是final的不是常量，所以我们在Library工程不能在switch - case 和Annotation中使用资源索引。由于引用的资源不是final的，所以Library的产物aar中包含的class中使用的资源索引还是会以包名存在。
			- ![image.png](../assets/image_1684423043172_0.png)
			- 在App工程中构建时会将依赖的AAR资源进行合并，根据合并的结果生成最终的R资源索引，这时的资源索引已经确定，所以全部是final的，java编译器在编译时会将final常量进行inline内联操作，也就是App工程中的java源码编译后的class中使用的R资源索引全部会替换为常量值。但resource.arsc文件中有关library的R资源类型还依然存在。
			- 我们采用了业界比较优秀的方案[ByteX R文件瘦身](https://github.com/bytedance/ByteX/blob/master/shrink-r-plugin/README-zh.md)。他是通过编译时使用Transform处理R文件的class，收集R资源相关的信息。流程如下：
			- 首先默认当前R文件class是可以被删除的。
			  如果是静态final int的field并且没有在白名单中则添加到shouldBeInlinedRFields集合中，否则添加到shouldSkipInlineRFields集合中，标记当前class不能被删除。
			  如果当前R文件所有field没有在白名单中则添加到shouldDiscardRClasses集合中，也就是该R文件class是可以被删除的。
			  如果扫描到Styleable class的方法时，方法用于初始化静态变量和静态代码块，并且不需要保留该类时，会使用AnalyzeStyleableClassVisitor处理，如果数组大小和计算的大小一致则加到shouldBeInlinedRFields集合，否则抛出异常。
			  真正开始Transform处理R文件，扫描filed判断进行删除
			  58app在使用ByteX将R资源inline后，包大小减少了4.6M，dex数量从16个减到了11个。
		- ## [[ 图片压缩、使用WebP等]]
		- ## [[语言配置只保留中文]]
		- ## [[非全屏图片只保留一套]]
		- ## [[单架构so，同城配置的v7a]]
		- ## 还有哪些措施
			- 网络加载图片，调研过但因无法统计失败情况而放弃
			- 无用代码统计移除，工作量大，需要长时间统计数据后分析
			- 动态加载so文件
			- Facebook的Redex：将dex重新分包
			- assets文件夹内优化
		- # 包大小监控
		  collapsed:: true
			- ## 手动统计
			  collapsed:: true
				- 同城APP的包大小监控在每次版本发布后都会以邮件的方式进行展示各模块大小。
					- ![image.png](../assets/image_1684423350862_0.png)
					- ![image.png](../assets/image_1684423359895_0.png)
					- 在很早之前的版本都是手动统计各个模块大小，手动统计不仅费时费力，有时也会面临着统计不够全面的情况。
		- ## Zucker自动化
		  collapsed:: true
			- 19年底我们就计划将人力从繁琐的手动统计AAR大小中解放出来，我们自研并开源了Zucker工具，这可能是业界第一个从模块角度自动化统计包大小的工具。在当时Zucker有以下特点：
			- 业界首款可以从模块角度统计APK大小的工具；
			- 利用用户自身的运行环境即可运行，具有一定的通用性；
			- 使用脚本即可实现，极小的入侵用户代码，灵活性高、扩展性强；
			- 对比原有的统计方法，通过自动化，实现提效；
			- Zucker的整体架构如下：自动化打包统计、依赖分析、目标AAR模拟：
				- ![image.png](../assets/image_1684423397498_0.png)
			- 但随着使用过程中，Zucker也暴露的一些问题：mock aar时会导致某些资源找不到，导致打包计算失败；打包时间虽相对手动统计有所提升，但时间上还是很长。
		- ## APS CI自动化
		  collapsed:: true
			- 与Zucker基于本项目打包计算AAR大小的思路不同，卫国创新性的使用了一个demo工程来统计AAR打入到apk中来计算AAR的大小，并修改了aapt2源码解决资源找不到的问题。这样使用APS能够解决Zucker的暴露所有问题，统计速度也更上一层楼。
			- ## APS流程
				- 首先执行打包获取demo包大小，
				- 然后获取依赖树列表，
				- 将依赖逐个添加到demo中并移除pom依赖，
				- 执行循环打包获取增量包大小，输出json结果。
				- ![image.png](../assets/image_1684423439265_0.png)
				- 为了方便使用，APS与我们的AVM CI系统整合，专门用于产出包大小报告，它会自动下载58app工程源码并获取依赖树，一键式的发送包大小邮件。
				- ![image.png](../assets/image_1684423457578_0.png)
		- 总结与展望
		- 58同城APP在包大小治理上做过很多的努力，总结下来就是对APK内的各个文件进行优化，有对于res文件夹内资源的压缩与适度删除，也有对于dex文件进行的混淆和去除R文件内联等操作，还有对于resource.arsc文件内进行混淆的业界做法。我们激进地探索过内置图片网络化的功能，也有对于无用代码删除的方案踌躇过，这些都是我们对于包大小治理的方方面面。
		- 但随着时间的推移工程的越来越庞大，业务功能不断丰富累牍，包大小治理道阻且长。
		- 参考：
		- https://ishare.58corp.com/articleDetail?id=77385
		- https://ishare.58corp.com/articleDetail?id=24157
		- https://ishare.58corp.com/articleDetail?id=60576
		- https://juejin.cn/post/6844904103131234311
		- https://booster.johnsonlee.io/zh/guide/shrinking/res-index-inlining.html#%E5%88%A0%E9%99%A4%E4%B8%8D%E5%BF%85%E8%A6%81%E7%9A%84-r
		- https://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=208135658&idx=1&sn=ac9bd6b4927e9e82f9fa14e396183a8f#rd
		- https://developer.android.google.cn/training/multiscreen/screendensities?hl=zh_cn