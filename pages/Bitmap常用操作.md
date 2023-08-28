# 1.将Bitmap保存为本地文件：
collapsed:: true
	- ```java
	      public static void writeBitmapToFile(String filePath, Bitmap b, int quality)
	      {
	          try {
	              File desFile = new File(filePath);
	              FileOutputStream fos = new FileOutputStream(desFile);
	              BufferedOutputStream bos = new BufferedOutputStream(fos);
	              b.compress(Bitmap.CompressFormat.JPEG, quality, bos);
	              bos.flush();
	              bos.close();
	          } catch (IOException e) {
	              e.printStackTrace();
	          }
	      }
	  ```
- # 2.图片压缩：
  collapsed:: true
	- ```java
	      private static Bitmap compressImage(Bitmap image) {
	          if (image == null) {
	              return null;
	          }
	          ByteArrayOutputStream baos = null;
	          try {
	              baos = new ByteArrayOutputStream();
	              image.compress(Bitmap.CompressFormat.JPEG, 50, baos);
	              byte[] bytes = baos.toByteArray();
	              ByteArrayInputStream isBm = new ByteArrayInputStream(bytes);
	              Bitmap bitmap = BitmapFactory.decodeStream(isBm);
	              return bitmap;
	          } catch (OutOfMemoryError e) {
	          } finally {
	              try {
	                  if (baos != null) {
	                      baos.close();
	                  }
	              } catch (IOException e) {
	              }
	          }
	          return null;
	      }
	  ```
- # 3.图片缩放：
	- ```java
	      /**
	       * 根据scale生成一张图片
	       *
	       * @param bitmap
	       * @param scale 等比缩放值
	       * @return
	       */
	      public static Bitmap bitmapScale(Bitmap bitmap, float scale) {
	          Matrix matrix = new Matrix();
	          matrix.postScale(scale, scale); // 长和宽放大缩小的比例
	          Bitmap resizeBmp = Bitmap.createBitmap(bitmap, 0, 0, bitmap.getWidth(),
	                  bitmap.getHeight(),
	                  matrix, true);
	          return resizeBmp;
	      }
	  ```
- # 4.获取图片旋转角度：
	- ```java
	      /**
	       * 读取照片exif信息中的旋转角度
	       *
	       * @param path 照片路径
	       * @return角度
	       */
	      private static int readPictureDegree(String path) {
	          if (TextUtils.isEmpty(path)) {
	              return 0;
	          }
	          int degree = 0;
	          try {
	              ExifInterface exifInterface = new ExifInterface(path);
	              int orientation =
	                      exifInterface.getAttributeInt(ExifInterface.TAG_ORIENTATION,
	                              ExifInterface.ORIENTATION_NORMAL);
	              switch (orientation) {
	                  case ExifInterface.ORIENTATION_ROTATE_90:
	                      degree = 90;
	                      break;
	                  case ExifInterface.ORIENTATION_ROTATE_180:
	                      degree = 180;
	                      break;
	                  case ExifInterface.ORIENTATION_ROTATE_270:
	                      degree = 270;
	                      break;
	              }
	          } catch (Exception e) {
	          }
	          return degree;
	      }
	  ```
- # 5.设置图片旋转角度
	- ```java
	      private static Bitmap rotateBitmap(Bitmap b, float rotateDegree) {
	          if (b == null) {
	              return null;
	          }
	          Matrix matrix = new Matrix();
	          matrix.postRotate(rotateDegree);
	          Bitmap rotaBitmap = Bitmap.createBitmap(b, 0, 0, b.getWidth(),
	                  b.getHeight(), matrix, true);
	          return rotaBitmap;
	      }
	  ```
- # 6.通过图片id获得Bitmap:
	- ```java
	  Bitmap bitmap=BitmapFactory.decodeResource(getResources(),
	  R.drawable.ic_launcher);
	  ```
- # 7.通过 assest 获取 获得Drawable bitmap
	- ```java
	  InputStream in = this.getAssets().open("ic_launcher");
	  Drawable da = Drawable.createFromStream(in, null);
	  Bitmap mm = BitmapFactory.decodeStream(in);
	  ```
- # 8.通过 sdcard 获得 bitmap
	- ```java
	  Bitmap bit = BitmapFactory.decodeFile("/sdcard/1 android.jpg");
	  ```
- # 9.view转Bitmap
	- ```java
	      public static Bitmap convertViewToBitmap(View view, int bitmapWidth, int
	              bitmapHeight){
	          Bitmap bitmap = Bitmap.createBitmap(bitmapWidth, bitmapHeight,
	                  Bitmap.Config.ARGB_8888);
	          view.draw(new Canvas(bitmap));
	          return bitmap;
	      }
	  ```
- # 10.将控件转换为bitmap
  collapsed:: true
	- ```java
	      public static Bitmap convertViewToBitMap(View view){
	  // 打开图像缓存
	          view.setDrawingCacheEnabled(true);
	  // 必须调用measure和layout方法才能成功保存可视组件的截图到png图像文件
	  // 测量View大小
	          view.measure(MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED),
	                  MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED));
	  // 发送位置和尺寸到View及其所有的子View
	          view.layout(0, 0, view.getMeasuredWidth(), view.getMeasuredHeight());
	  // 获得可视组件的截图
	          Bitmap bitmap = view.getDrawingCache();
	          return bitmap;
	      }
	  
	      public static Bitmap getBitmapFromView(View view){
	          Bitmap returnedBitmap = Bitmap.createBitmap(view.getWidth(),
	                  view.getHeight(), Bitmap.Config.ARGB_8888);
	          Canvas canvas = new Canvas(returnedBitmap);
	          Drawable bgDrawable = view.getBackground();
	          if (bgDrawable != null)
	              bgDrawable.draw(canvas);
	          else
	              canvas.drawColor(Color.WHITE);
	          view.draw(canvas);
	          return returnedBitmap;
	      }
	  ```
- # 11.放大缩小图片
	- ```java
	      public static Bitmap zoomBitmap(Bitmap bitmap,int w,int h){
	          int width = bitmap.getWidth();
	          int height = bitmap.getHeight();
	          Matrix matrix = new Matrix();
	          float scaleWidht = ((float)w / width);
	          float scaleHeight = ((float)h / height);
	          matrix.postScale(scaleWidht, scaleHeight);
	          Bitmap newbmp = Bitmap.createBitmap(bitmap, 0, 0, width, height, matrix,
	                  true);
	          return newbmp;
	      }
	  ```
- # 12.获得圆角图片的方法
	- ```java
	   public static Bitmap getRoundedCornerBitmap(Bitmap bitmap,float roundPx){
	          Bitmap output = Bitmap.createBitmap(bitmap.getWidth(), bitmap
	                  .getHeight(), Config.ARGB_8888);
	          Canvas canvas = new Canvas(output);
	          final int color = 0xff424242;
	          final Paint paint = new Paint();
	          final Rect rect = new Rect(0, 0, bitmap.getWidth(), bitmap.getHeight());
	          final RectF rectF = new RectF(rect);
	          paint.setAntiAlias(true);
	          canvas.drawARGB(0, 0, 0, 0);
	          paint.setColor(color);
	          canvas.drawRoundRect(rectF, roundPx, roundPx, paint);
	          paint.setXfermode(new PorterDuffXfermode(Mode.SRC_IN));
	          canvas.drawBitmap(bitmap, rect, rect, paint);
	          return output;
	      }
	  ```
- # 13.对 bitmap 进行裁剪
	- ```java
	  
	      public Bitmap bitmapClip(Context context , int id , int x , int y){
	          Bitmap map = BitmapFactory.decodeResource(context.getResources(), id);
	          map = Bitmap.createBitmap(map, x, y, 120, 120);
	          return map;
	      }
	  ```