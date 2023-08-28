- ```java
  public void recycle() // 回收位图占用的内存空间，把位图标记为Dead
  public final boolean isRecycled() //判断位图内存是否已释放
  public final int getWidth() //获取位图的宽度
  public final int getHeight() //获取位图的高度
  public final boolean isMutable() //图片是否可修改
  public int getScaledWidth(Canvas canvas) //获取指定密度转换后的图像的宽度
  public int getScaledHeight(Canvas canvas) //获取指定密度转换后的图像的高度
  public boolean compress(CompressFormat format, int quality, OutputStream
  stream) //按指定的图片格式以及画质，将图片转换为输出流
  public static Bitmap createBitmap(Bitmap src) //以src 为原图生成不可变得新图像
  
   //以src 为原图，创建新的图像，指定新图像的高宽以及是否可变。
  public static Bitmap createScaledBitmap(Bitmap src, int dstWidth, int
  dstHeight, boolean filter)
  
  //创建指定格式、大小的位图
  public static Bitmap createBitmap(int width, int height, Config config) 
  
  //以source 为原图，创建新的图片，指定起始坐标以及新图像的高宽。  
  public static Bitmap createBitmap(Bitmap source, int x, int y, int width, int
  height) 
  ```