## 1、会不会在R文件生成索引id不同
	- ## res：会在R文件里生成索引ID，
	- ## assets：不会在R文件里生成索引id
- ## 2、访问方式不同
	- ### res用getResource()访问
	- ### assets用AssetsManager访问
- ## 3、会不会被二进制编译
	- [[#red]]==**res/raw与assets**==里的文件在打包的时候==**都不会被系统二进制编译，都被原封不动打包进APK**==
	- res/xml[[#red]]==**会被编译成二进制文件，没用到的不会打进apk**==
- ## 4、存储资源不同
	- res/anim存放动画资源
	- assets通常用来存放游戏资源、脚本、字体文件等。
- ## 5、能否创建子文件夹不同
	- res/raw不可以创建子文件夹
	- assets可以。
-
- ![区别](https://img-blog.csdn.net/20180212084740468?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGljaGFveGluMTY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
  id:: 64e8ba43-45ee-48d3-ae9b-b3e69f7beb27
-