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
-