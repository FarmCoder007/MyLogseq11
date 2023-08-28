# 1.Drawable转化成Bitmap
collapsed:: true
	- ```java
	      public static Bitmap drawableToBitmap(Drawable drawable) {
	          Bitmap bitmap = Bitmap.createBitmap(drawable.getIntrinsicWidth(),
	                  drawable.getIntrinsicHeight(),
	                  drawable.getOpacity() != PixelFormat.OPAQUE ?
	                          Bitmap.Config.ARGB_8888 : Bitmap.Config.RGB_565);
	          Canvas canvas = new Canvas(bitmap);
	          drawable.setBounds(0, 0, drawable.getIntrinsicWidth(),
	                  drawable.getIntrinsicHeight());
	          drawable.draw(canvas);
	          return bitmap;
	      }
	  ```
	- drawable 的获取方式： Drawable drawable = getResources().getDrawable(R.drawable.ic_launcher);
- # 2.Bitmap转换成Drawable
	- ```java
	  public static Drawable bitmapToDrawable(Resources resources, Bitmap bm) {
	      Drawable drawable = new BitmapDrawable(resources, bm);
	      return drawable;
	  }
	  ```
- # 3.Bitmap转换成byte[]
	- ```java
	      public byte[] bitmap2Bytes(Bitmap bm) {
	          ByteArrayOutputStream baos = new ByteArrayOutputStream();
	          bm.compress(Bitmap.CompressFormat.PNG, 100, baos);
	          return baos.toByteArray();
	      }
	  ```
- # 4.byte[]转换成Bitmap
	- ```java
	  Bitmap bitmap = BitmapFactory.decodeByteArray(1 byte, 0, b.length);
	  ```
- # 5.InputStream转换成Bitmap
	- ```java
	  InputStream is = getResources().openRawResource(id);
	  Bitmap bitmap = BitmaoFactory.decodeStream(is);
	  ```
- # 6.InputStream转换成byte[]
	- ```java
	  		//也可以通过其他方式接收一个 InputStream对象
	  		InputStream is = getResources().openRawResource(id);
	          ByteArrayOutputStream baos = new ByteArrayOutputStream();
	          byte[] b = new byte[1024*2];
	          int len = 0;
	          while ((len = is.read(b, 0, b.length)) != -1)
	          {
	          baos.write(b, 0, len);
	          baos.flush();
	          }
	          byte[] bytes = baos.toByteArray();
	  ```
-