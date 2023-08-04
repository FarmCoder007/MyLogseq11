- ```java
  public class WrapContentHeightViewPager extends ViewPager {
     
     public WrapContentHeightViewPager(@NonNull Context context) {
         super(context);
     }
  
     public WrapContentHeightViewPager(@NonNull Context context, @Nullable AttributeSet attrs) {
         super(context, attrs);
     }
  
     @Override
     protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
         int height = 0;
         for (int i = 0; i < getChildCount(); i++) {
             View child = getChildAt(i);
             // 错误
             child.measure(widthMeasureSpec, MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED));
             int h = child.getMeasuredHeight();
             if (h > height) height = h;
         }
  
         heightMeasureSpec = MeasureSpec.makeMeasureSpec(height, MeasureSpec.EXACTLY);
         super.onMeasure(widthMeasureSpec, heightMeasureSpec);
     }
  }
  ```