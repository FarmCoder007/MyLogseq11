# Option 参数类
	- ## public boolean inJustDecodeBounds
		- //如果设置为true ，不获取图片，不分配内存，但会
		  返回图片的高度宽度信息。如果将这个值置为true ，那么在解码的时候将不会返回bitmap ，只
		  会返回这个bitmap 的尺寸。这个属性的目的是，如果[[#red]]==**你只想知道一个bitmap 的尺寸，但又不想
		  将其加载到内存时。这是一个非常有用的属性**==。
	- ## public int inSampleSize //图片缩放的倍数，
	  collapsed:: true
		- 这个值是一个int
		- 当它小于1的时候，将会被当做1处理
		- 如果大于1，那么就会按照比例（1 / inSampleSize） 缩小bitmap 的宽和高、降
		  低分辨率，大于1时这个值将会被处置为2的倍数。例如， width=100，height=100，
		  inSampleSize=2 ，那么就会将bitmap 处理为， width=50，height=50 ，宽高降为1 / 2，像素数
		  降为1 / 4。
	- ## public int outWidth //获取图片的宽度值
	- ## public int outHeight //获取图片的高度值
		- 表示这个Bitmap 的宽和高，一般和inJustDecodeBounds 一起使用来获得Bitmap 的宽高，但是不加载到内存。
	- ## public int inDensity //用于位图的像素压缩比
	- ## public int inTargetDensity
		- //用于目标位图的像素压缩比（要生成的位图）
	- ## public byte[] inTempStorage //创建临时文件，将图片存储
	- ## public boolean inScaled
		- //设置为true 时进行图片压缩，从inDensity 到 inTargetDensity
	- ## public boolean inDither //如果为true ,解码器尝试抖动解码
	- ## public Bitmap.Config inPreferredConfig //设置解码器，
		- 这个值是设置色彩模式，默认值
		  是ARGB_8888 ，在这个模式下，一个像素点占用4bytes空间，一般对透明度不做要求的话，一般
		  采用RGB_565 模式，这个模式下一个像素点占用2bytes。
	- ## public String outMimeType
		- //设置解码图像
	- ## public boolean inPurgeable
		- //当存储Pixel 的内存空间在系统内存不足时是否可以被回收
	- ## public boolean inInputShareable
		- // inPurgeable 为true 情况下才生效，是否可以共享一个InputStream
	- ## public boolean inPreferQualityOverSpeed
		- //为true 则优先保证Bitmap 质量其次是解码速度
	- ## public boolean inMutable
		- //配置Bitmap 是否可以更改
		- 比如：在Bitmap 上隔几个像素加一条线段
	- ## public int inScreenDensity
		- //当前屏幕的像素密度
- # 工厂方法
	- ```java
	  //从文件读取图片
	  public static Bitmap decodeFile(String pathName, Options opts) 
	  public static Bitmap decodeFile(String pathName)
	    
	  //从输入流读取图片  
	  public static Bitmap decodeStream(InputStream is) 
	  public static Bitmap decodeStream(InputStream is, Rect outPadding, Options
	  opts)
	    
	  //从资源文件读取图片  
	  public static Bitmap decodeResource(Resources res, int id) 
	  public static Bitmap decodeResource(Resources res, int id, Options opts)
	    
	  //从数组读取图片
	  public static Bitmap decodeByteArray(byte[] data, int offset, int length) 
	  public static Bitmap decodeByteArray(byte[] data, int offset, int length,
	  Options opts)
	    
	  //从文件读取文件 与decodeFile 不同的是这个直接调用JNI函数进行读取 效率比较高  
	  public static Bitmap decodeFileDescriptor(FileDescriptor fd) 
	  public static Bitmap decodeFileDescriptor(FileDescriptor fd, Rect outPadding,
	  Options opts)
	  ```