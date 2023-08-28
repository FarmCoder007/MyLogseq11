- 1、res会在R.java生成索引ID，在打包的时候判断资源有没有用到，没用到的时候不会被打包进apk中(res/raw文件夹除外)，
	- 而assets不会生成索引id,不会被系统二进制编译，都被原封不动打包进APK
- 2.、res用getResource()访问，assets用AssetsManager访问。
- 3.、res/raw与assets里的文件在打包的时候都不会被系统二进制编译，都被原封不动打包进APK，assets通常用来存放游戏资源、脚本、字体文件等。
	- 但res/raw不可以创建子文件夹，而assets可以。
- 4、res/xml会被编译成二进制文件。res/anim存放动画资源。
-
- ![区别](https://img-blog.csdn.net/20180212084740468?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGljaGFveGluMTY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
  id:: 64e8ba43-45ee-48d3-ae9b-b3e69f7beb27
-