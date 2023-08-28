- Bitmap 的加载方式有Resource 资源加载、本地（SDcard）加载、网络加载等加载方式。
- ## 1. 从本地（SDcard）文件读取
	- ### 方式一
	  collapsed:: true
		- ```java
		      /**
		       * 获取缩放后的本地图片
		       *
		       * @param filePath 文件路径
		       * @param width 宽
		       * @param height 高
		       * @return
		       */
		      public static Bitmap readBitmapFromFile(String filePath, int width, int height) {
		          BitmapFactory.Options options = new BitmapFactory.Options();
		          options.inJustDecodeBounds = true;
		          BitmapFactory.decodeFile(filePath, options);
		          float srcWidth = options.outWidth;
		          float srcHeight = options.outHeight;
		          int inSampleSize = 1;
		          if (srcHeight > height || srcWidth > width) {
		              if (srcWidth > srcHeight) {
		                  inSampleSize = Math.round(srcHeight / height);
		              } else {
		                  inSampleSize = Math.round(srcWidth / width);
		              }
		          }
		          options.inJustDecodeBounds = false;
		          options.inSampleSize = inSampleSize;
		          return BitmapFactory.decodeFile(filePath, options);
		      }
		  ```
	- ### 方式二：效率高于方式一
	  collapsed:: true
		- ```java
		      /**
		       * 获取缩放后的本地图片
		       *
		       * @param filePath 文件路径
		       * @param width 宽
		       * @param height 高
		       * @return
		       */
		      public static Bitmap readBitmapFromFileDescriptor(String filePath, int
		              width, int height) {
		          try {
		              FileInputStream fis = new FileInputStream(filePath);
		              BitmapFactory.Options options = new BitmapFactory.Options();
		              options.inJustDecodeBounds = true;
		              BitmapFactory.decodeFileDescriptor(fis.getFD(), null, options);
		              float srcWidth = options.outWidth;
		              float srcHeight = options.outHeight;
		              int inSampleSize = 1;
		              if (srcHeight > height || srcWidth > width) {
		                  if (srcWidth > srcHeight) {
		                      inSampleSize = Math.round(srcHeight / height);
		                  } else {
		                      inSampleSize = Math.round(srcWidth / width);
		                  }
		              }
		              options.inJustDecodeBounds = false;
		              options.inSampleSize = inSampleSize;
		              return BitmapFactory.decodeFileDescriptor(fis.getFD(), null,
		                      options);
		          } catch (Exception ex) {
		          }
		          return null;
		      }
		  ```
- ## 2. 从输入流中读取文件（网络加载）
  collapsed:: true
	- ```java
	      /**
	       * 获取缩放后的本地图片
	       *
	       * @param ins 输入流
	       * @param width 宽
	       * @param height 高
	       * @return
	       */
	      public static Bitmap readBitmapFromInputStream(InputStream ins, int width,
	                                                     int height) {
	          BitmapFactory.Options options = new BitmapFactory.Options();
	          options.inJustDecodeBounds = true;
	          BitmapFactory.decodeStream(ins, null, options);
	          float srcWidth = options.outWidth;
	          float srcHeight = options.outHeight;
	          int inSampleSize = 1;
	          if (srcHeight > height || srcWidth > width) {
	              if (srcWidth > srcHeight) {
	                  inSampleSize = Math.round(srcHeight / height);
	              } else {
	                  inSampleSize = Math.round(srcWidth / width);
	              }
	          }
	          options.inJustDecodeBounds = false;
	          options.inSampleSize = inSampleSize;
	          return BitmapFactory.decodeStream(ins, null, options);
	      }
	  ```
- ## 3.Resource资源加载
  collapsed:: true
	- Res资源加载方式：
	  collapsed:: true
		- ```java
		      public static Bitmap readBitmapFromResource(Resources resources, int
		              resourcesId, int width, int height) {
		          BitmapFactory.Options options = new BitmapFactory.Options();
		          options.inJustDecodeBounds = true;
		          BitmapFactory.decodeResource(resources, resourcesId, options);
		          float srcWidth = options.outWidth;
		          float srcHeight = options.outHeight;
		          int inSampleSize = 1;
		          if (srcHeight > height || srcWidth > width) {
		              if (srcWidth > srcHeight) {
		                  inSampleSize = Math.round(srcHeight / height);
		              } else {
		                  inSampleSize = Math.round(srcWidth / width);
		              }
		          }
		          options.inJustDecodeBounds = false;
		          options.inSampleSize = inSampleSize;
		          return BitmapFactory.decodeResource(resources, resourcesId, options);
		      }
		  ```
	- 此种方式相当的耗费内存 建议采用decodeStream 代替decodeResource 可以如下形式:
	  collapsed:: true
		- ```java
		      public static Bitmap readBitmapFromResource(Resources resources, int
		              resourcesId, int width, int height) {
		          InputStream ins = resources.openRawResource(resourcesId);
		          BitmapFactory.Options options = new BitmapFactory.Options();
		          options.inJustDecodeBounds = true;
		          BitmapFactory.decodeStream(ins, null, options);
		          float srcWidth = options.outWidth;
		          float srcHeight = options.outHeight;
		          int inSampleSize = 1;
		          if (srcHeight > height || srcWidth > width) {
		              if (srcWidth > srcHeight) {
		                  inSampleSize = Math.round(srcHeight / height);
		              } else {
		                  inSampleSize = Math.round(srcWidth / width);
		              }
		          }
		          options.inJustDecodeBounds = false;
		          options.inSampleSize = inSampleSize;
		          return BitmapFactory.decodeStream(ins, null, options);
		      }
		  ```
	-
	- BitmapFactory.decodeResource 加载的图片可能会经过缩放，该缩放目前是放在 java 层做的，效率
	  比较低，而且需要消耗 java 层的内存。因此，如果大量使用该接口加载图片，容易导致OOM错误
	- BitmapFactory.decodeStream 不会对所加载的图片进行缩放，相比之下占用内存少，效率更高。
	- 这两个接口各有用处，如果对性能要求较高，则应该使用 decodeStream ；
	- 如果对性能要求不高，且需要 Android 自带的图片自适应缩放功能，则可以使用 decodeResource
- ## 4.Assets资源加载方式：
  collapsed:: true
	- ```java
	      /**
	       * 获取缩放后的本地图片
	       *
	       * @param filePath 文件路径,即文件名称
	       * @return
	       */
	      public static Bitmap readBitmapFromAssetsFile(Context context, String
	              filePath) {
	          Bitmap image = null;
	          AssetManager am = context.getResources().getAssets();
	          try {
	              InputStream is = am.open(filePath);
	              image = BitmapFactory.decodeStream(is);
	              is.close();
	          } catch (IOException e) {
	              e.printStackTrace();
	          }
	          return image;
	      }
	  ```
- ## 5.从二进制数据读取图片
  collapsed:: true
	- ```java
	      public static Bitmap readBitmapFromByteArray(byte[] data, int width, int
	              height) {
	          BitmapFactory.Options options = new BitmapFactory.Options();
	          options.inJustDecodeBounds = true;
	          BitmapFactory.decodeByteArray(data, 0, data.length, options);
	          float srcWidth = options.outWidth;
	          float srcHeight = options.outHeight;
	          int inSampleSize = 1;
	          if (srcHeight > height || srcWidth > width) {
	              if (srcWidth > srcHeight) {inSampleSize = Math.round(srcHeight / height);
	              } else {
	                  inSampleSize = Math.round(srcWidth / width);
	              }
	          }
	          options.inJustDecodeBounds = false;
	          options.inSampleSize = inSampleSize;
	          return BitmapFactory.decodeByteArray(data, 0, data.length, options);
	      }
	  ```