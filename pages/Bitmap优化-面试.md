# 1、怎么获取Bitmap尺寸#card
collapsed:: true
	- ## BitmapFactory的options
		- ## public boolean inJustDecodeBounds
			- //如果设置为true ，不获取图片，不分配内存，但会
			  返回图片的高度宽度信息。如果将这个值置为true ，那么在解码的时候将不会返回bitmap ，只
			  会返回这个bitmap 的尺寸。这个属性的目的是，如果[[#red]]==**你只想知道一个bitmap 的尺寸，但又不想
			  将其加载到内存时。这是一个非常有用的属性**==。
		- ## public int outWidth //获取图片的宽度值
		- ## public int outHeight //获取图片的高度值
		  collapsed:: true
			- 表示这个Bitmap 的宽和高，一般和inJustDecodeBounds 一起使用来获得Bitmap 的宽高，但是不加载到内存。
- # 2、Bitmap内存模型#card
	- Bitmap对象存放在 java  heap堆中
	- 像素数据  存放在 Native heap中
- # 3. 获取Bitmap大小api :getByteCount()与getAllocationByteCount()的区别#card
	- 一般情况下两者是相等的
	- 通过[[#green]]==**复用Bitmap来解码图片，**==如果被复用的Bitmap的内存比待分配内存的Bitmap大,那么
	- getByteCount()表示[[#red]]==**新解码图片占用内存的大小**==（并非实际内存大小,实际大小是复用的那个
	  Bitmap的大小），
	- getAllocationByteCount()表示[[#red]]==**被复用Bitmap真实占用的内存大小**==（即mBuffer
	  的长度）
	- 参考：[[获取Bitmap的大小]]的API
- # 4、怎么计算Bitmap占的内存大小#card
	- ## 参考
		- Bitamp 占用内存大小 = 宽度像素 x （inTargetDensity / inDensity） x 高度像素 x （inTargetDensity / inDensity）x 一个像素所占的内存
		- 注：这里inDensity表示目标图片的dpi（放在哪个资源文件夹下），inTargetDensity表示目标屏幕的dpi，所以你可以发现inDensity和inTargetDensity会对Bitmap的宽高
		  进行拉伸，进而改变Bitmap占用内存的大小。
		- 在Bitmap里有两个获取内存占用大小的方法。
	- >Bitmap内存占用 ≈ 像素数据总大小 = 图片宽 × 图片高× (设备屏幕密度/资源目录屏幕密度)^2 × 每个像素的字节大小
	- ## [[单个像素的字节大小]]RGB-565 2字节，ARGB888 1像素4字节
	- 1. 图片放在drawable中，等同于放在drawable-mdpi中，原因为：drawable目录不具有屏幕密度特
	  性，所以采用基准值，即mdpi
	- 2. 图片放在某个特定drawable中，比如drawable-hdpi，如果设备的屏幕密度高于当前drawable目
	  录所代表的密度，则图片会被放大，否则会被缩小
	  放大或缩小比例 = 设备屏幕密度 / drawable目录所代表的屏幕密度
- # 5、Bitmap加载大图片怎么分块加载#card
	- ## BitmapRegionDecoder，具体没用过
- # 6、bitmap 和 drawable的区别?#card
	- ## Bitmap
		- 是把实际像素数据，映射到内存的对象里【就是  图片信息的存储工具】
	- ## Drawable是绘制工具
		- 他是一个可以调用canvas 的绘制方法
- # 7、bitmap和Drawable互转原理？
	- ## bitmap转Drawable: 图形存储工具 转绘制工具
		- 代码的实现原理:  创建一个bitmapDrawable    把 bitmap传入  在BitmapDrawable 的draw方法 里把bitmap 绘制一遍
	- ## Drawable 转bitmap：绘制规则 转像素信息
		- 原理：创建一个新的bitmap    然后 用这个Drawable 像素规则  画在这个bitmap上
-