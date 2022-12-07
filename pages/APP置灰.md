- ```java
  public class GrayManager {
  
      private static GrayManager mInstance;
      private Paint mGrayPaint;
      private ColorMatrix mGrayMatrix;
  
      public static GrayManager getInstance() {
          if (mInstance == null) {
              synchronized (GrayManager.class) {
                  if (mInstance == null) {
                      mInstance = new GrayManager();
                  }
              }
          }
          return mInstance;
      }
  
      //初始化
      public void init() {
          mGrayMatrix = new ColorMatrix();
          mGrayPaint = new Paint();
          mGrayMatrix.setSaturation(0);
          mGrayPaint.setColorFilter(new ColorMatrixColorFilter(mGrayMatrix));
      }
  
  
      //硬件加速置灰方法
      public void setLayerGrayType(View view) {
          if (mGrayMatrix == null || mGrayPaint == null) {
              init();
          }
  
          view.setLayerType(View.LAYER_TYPE_HARDWARE, mGrayPaint);
      }
  }
  
  ```